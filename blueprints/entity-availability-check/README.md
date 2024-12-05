# monitor-entity-availability

## Resume

Home Assistant BluePrints usefull to build automations to check if Home Assistant entities are available or not.
If entities becomes unavailable, a notification will be sent with the list of unavailable entities.

Two blueprints can be used:
- monitor_entity_availability_with_label.yaml : check entities which have a specific label.
- monitor_entity_availability_with_wildcards.yaml : check entities which ID match with a string filter with wildcards.

## Details

Your home automation system is at the heart of your home, and potentially manages critical functions: heating, ventilation systems, fire sensors or alarms. But what happens if devices become unresponsive? The consequences can be unfortunate.

This blueprints offers you to monitor your various components, through the verification of the availability of critical entities.

### Goal
A BluePrint will allow you to create one or more automations that will, at regular intervals, check your critical entities. If one of the entities becomes unavailable, then the automation will send you a notification.

The main challenge is to identify all the entities to be monitored. We offer **2 methods to choose from**:

1. **Method 1: Using Labels**
Starting with Home Assistant version 2024.4, it is possible to associate [labels](https://www.home-assistant.io/docs/organizing/labels/) with entities. We propose to associate the critical entities to be monitored with a specific label ("check" for example).
2. **Method 2: Using Expressions with Wildcards**
If your entities are correctly named, then it is possible to list them with just a string and **wildcards**, as one would do in any search (in a file manager for example, by putting wildcards "*" in the file search field).

> ⚠️
> **Method 1 (labels)** is very flexible and the simplest, but requires you to associate a label with all of your entities to monitor.
>
> **Method 2 (wildcard)** does not require tagging, but assumes that you have a clean and standardized naming of your entities, which is always strongly recommended.
>
> Everyone will choose the method they deem most appropriate (or even the 2), depending on their system, the number of entities and the quality of their namings.

### Method 1 - Selection by labels
Method 1 proposes to monitor all entities that have a given label.

When creating the monitoring automation, the blueprint will ask you for several parameters:

The time(s) at which it is to run (up to 4 different hours during the day)
A label: the label you have used to "tag" all your critical entities.
Finally, the notification to be sent: the service to be called and the message to be sent, with the list of unavailable entities.

> ⚠️ You must have a Home Assistant version 2024.4 or higher.

#### Tag features
Go to Settings - Devices & Services and select the Entities tab.

Click on "Add a label" at the top right and create your label, for example "check". You can now associate the "check" tag with your critical entities.

Click on the selector in the banner.


Find and select the entities to monitor, then click on "Add a label" at the top right.

### Import the blueprint

In order to then be able to create an automation, I first need to import the blueprint from my Github repository.

Go to Settings, then Automations & Scenes, and finally click on the Blueprint tab.

Click on the button at the bottom right Import a blueprint.

Enter the blueprint address:

`(https://github.com/argonaute199/availability-check/blob/main/availability_check_label.yaml)`

That's it, the availability_check_label blueprint is imported and you can now create monitoring automations using it.

### Creating automation

Thanks to the BluePrint, we will be able to create an automation that triggers 1 to 4 times a day, checks the selected entities and sends a notification if one of the entities is not available.

Go to Settings, then Automations & Scenes, and finally stay on the Automations tab.

Click on the button at the bottom right Create an automation. You should see the BluePrint Availability check label.
![automation label](/media/entity-availability-check/image-4.png)

Click on it. You should then have access to the automation's settings.

- Complete 1 to 4 hours of execution during the day. Leave an hour field at zero if you don't want to use it. For example, fill in only "Hour 1" and "Hour 2" if you only want to run monitoring 2 times a day.
So you shouldn't start the automation at midnight, because the 0 hours are not taken into account...
- Specify the label. Copy the chosen label, e.g. check.
- Specify the notification to run.
Click Add Action
Choose an available notification. I use Telegram notifications (telegram bot: Send Message), but you can use the default Home Assistant notifications (persistante_notification:create).

Finally, in the message field, enter the code for a text of the type "Warning, devices not available:" and then in the {{entities}} line which at execution will be replaced by the list of the names of the unavailable entities.

> 💡 It is advisable to switch the field to YAML mode (click on the 3 dots to the right of the field for the menu) as below, and check that the message is the same as below.

![automation label details](/media/entity-availability-check/image-5.png)

Save your automation, it's created and ready to run. You can skip to the Testing chapter of the automation.

> 💡 Feel free to create 2 or 3 different automations that monitor groups of more or less critical entities. You will then need to create several labels (check1, check2...).
Critical entities will be monitored 4 times a day. Those that are less so only 1 to 2 times.

### Method 2 - wildcard selection

Method 2 proposes to select the entities to be monitored by defining a list of strings with "wildcards".

When creating the automation, the blueprint will ask you for several parameters:

- The time(s) at which it must run (up to 4 different hours during the day)
- A selection filter: a list of strings with "wildcards", which allows you to find all the entities to be monitored (see next chapter)
- Entities to exclude: entities that stand out in the filter, but that you want to exclude from monitoring.
- Finally, the notification to be sent: the service to be called and the message to be sent.

#### Selection filter - wildcards
The first step is to create a selection filter, i.e. a list of strings with "wildcards" that will filter the entities and find those that are to be monitored.

The wildcard is ".*" (a dot followed by a star).

**Example 1**

You want to return all entities related to the opening sensors.

For example, entities are "binary_sensors" whose name ends in "contact" (binary_sensors.aqara_office_window for example).

> => Window Detector Selection Filter
> **binary_sensor.*contact**

**Example 2**

You want to return all the entities related to the temperature sensors.

For example, entities are "sensors" whose name ends in "temperature". I usually put the manufacturer's name at the beginning and I have Oregon and Aqara. The temperatures of other devices don't interest me.

So I'm going to create a filter with 2 strings (one for aqara and one for oregon), separated by a comma.

> => Aqara and Oregon temperature sensor selection filter: sensor.aqara.*temperature, sensor.oregon.*temperature

> 💡 You can create a selection filter with as many comma-separated strings as you want.
> To select all the entities to monitor, it is likely that the filter will group 5 to 10 different channels.

#### Testing Selection Filters

It's important to test your selection filter and check what it actually returns.

To do this, go to Home Assistant in "development tools". Then click on the "Template" tab.

In the Template Editor field, copy the following code:

`{% set selector = 'sensor.*' %}

{% set result = namespace(entities=[]) %}
{% set entity_filter = selector.replace(' ', '').split(',') %}

{% for item in entity_filter %}
  {% for state in states | selectattr('entity_id', 'match', item)%}
    {% set result.entities = result.entities + [state.entity_id] %}
  {% endfor %}
{% endfor %}
{{result.entities|unique|join('\n')}}`

Then in the first line with the set selector code, replace the sensor.* code with your selection filter.

You will then get a list of the selected entities in the filter in the results box. In the following example, I find my temperature sensor entities to monitor.


### Import the blueprint
In order to then be able to create an automation, I first need to import the blueprint from my Github repository.

Go to Settings, then Automations & Scenes, and finally click on the Blueprint tab.

Click on the button at the bottom right Import a blueprint.

Enter the blueprint address:

`(https://github.com/argonaute199/availability-check/blob/main/availability_check_wildcards.yaml)`

That's it, the availability_check_wildcards blueprint is imported and you can now create monitoring automations using it.

### Creating automationç
Thanks to the BluePrint, we will be able to create an automation that triggers 1 to 4 times a day, checks the selected entities and sends a notification if an entity is not available.

Go to Settings, then Automations & Scenes, and finally stay on the Automations tab.

Click on the button at the bottom right Create an automation. You should see the BluePrint Availability check wildcards.
![create with wildcards](/media/entity-availability-check/image-6.png)

Click on it. You should then have access to the automation's settings.

- Complete 1 to 4 hours of execution during the day. Leave a time field at zero if you don't want to use it. For example, fill in only "Hour 1" and "Hour 2" if you only want to run monitoring 2 times a day.
So you shouldn't start the automation at midnight, because the 0 hours are not taken into account...
- Specify the filter. Copy the previously created string, allowing you to filter the entities to be excluded.
- Specify the entities to exclude. If your filter returns features that you don't want to monitor, specify which features to exclude. Click on Choose an entity, then find and select the entity...
- Specify the notification to run.
Click Add Action
Choose an available notification. I use Telegram notifications (telegram bot: Send Message), but you can use the default Home Assistant notifications (persistante_notification:create).

Finally, in the message field, enter the code for a text of the type "Warning, devices not available:" and then in the {{entities}} line which at execution will be replaced by the list of the names of the unavailable entities.
![configure blueprints wildcards](/media/entity-availability-check/image-33.png)

⚠️
> **Entering the notification message**
> There seems to be a pb that prohibits writing the full message with the UI.
> Start creating the call to the notification in UI mode with a bogus message. Then switch to YAML (see below) to finalize the writing of the full message.
![configure blueprints wildcards yaml](/media/entity-availability-check/image-11.png)

Save your automation, it's created and ready to run.

💡
Feel free to create 2 or 3 different automations, monitoring groups of more or less critical entities. Critical entities will be monitored 4 times a day. Those that are less so only 1 to 2 times.

### Testing automation

With your automation ready, it's a good idea to test it. With the automation in edit mode, click on the 3 dots at the top right and select "run".


The automation will run immediately. You should then have a notification with the warning message. If no entities are unavailable, the list will be empty during testing.

> ⚠️ **Be careful**, when running manually from the automation editor, the condition on the number of unavailable entities is not executed: the notification will be displayed with an empty list even if there are no unavailable entities (useful for testing).

> Of course, in normal execution (hourly triggering), the condition will apply and there will be a notification only if there are one or more unavailable entities.

To test, you can disable one of the monitored devices (a smart plug, a thermometer...) and wait for the associated entity to become unavailable. If you restart the run, this time you should get a notification displaying the name of the unavailable entity.

If you have any doubts, still in the automation editor, you can click on "run history" to check how the automation was executed.

If you click on the tab The "automation configuration" tab, you will find the full code generated by the blueprint. Finally, the "blueprint configuration" tab will return only the executed code (here the notification).

### Display in the dashboard
As an added bonus, if you use method 1 (labels), you can also easily create a map that displays in the dashboard if there are any unavailable features. For information, I use the term "device" in the display because we generally monitor one entity per device or "device"...

Here's the result if there are multiple unavailable entities:
![Multiple unavailable entities](/media/entity-availability-check/ecran-devices-indispos.png)

And if there is none:
![no unavailable entity](/media/entity-availability-check/ecran-no-devices-indispo-1.png)

To do this, you need to go to the editing dashboard and create a markdown card.
![markdown card](/media/entity-availability-check/carte-markdown.png)

Copy the following jinja2 code into the content field:

`{% set selector = 'check' %}
{% set result = namespace(entities=[]) %}
{% set entity_list = label_entities(selector)%}

{% for item in entity_list %}
  {% for state in states | selectattr('state','match', 'unavailable') | selectattr('entity_id', 'match', item)%}
    {% set result.entities = result.entities + [state.name] %}
  {% endfor %}
{% endfor %}

{% if result.entities | length == 0 %}
  **Aucun device indisponible.**
{% else %}
  {% if result.entities | length == 1 %}
    **Device indisponible :**
  {% else %}
    **Devices indisponibles :**
  {% endif %}
  {{ '\r\n - ' ~ result.entities|unique|join('\r\n - ')}}
{% endif %}`

Change the label in the first line if you are using a different one (check used in the code). Each time you visit the view, the map will be updated...

>💡It is quite possible to draw inspiration from this code for method 2 (wildcards), but I didn't.

### Conclusion

Setting up one or more automations that monitor your entities will help make your home automation system much more robust.


Full documentation is here (in french) : https://www.hacf.fr/surveillance-entites/