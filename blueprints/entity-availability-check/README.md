# monitor-entity-availability
Home Assistant BluePrints usefull to build automations to check if Home Assistant entities are available or not.
If entities becomes unavailable, a notification can be sent with the list of unavailable entities.

Two blueprints can be used:
- monitor_entity_availability_with_label.yaml : check entities which have a specific label.
- monitor_entity_availability_with_wildcards.yaml : check entities which ID match with a string filter with wildcards.

Full documentation is here (in french) : https://www.hacf.fr/surveillance-entites/