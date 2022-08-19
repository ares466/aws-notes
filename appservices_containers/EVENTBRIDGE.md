# EventBridge

EventBridge is an event bus that facilitates the creation of event-based architectures.

> NOTE: CloudWatch Events used to be the default event bus in AWS. EventBridge has replaced that service and should be used over CloudWatch Events.

Every account has a `default EventBridge event bus`. EventBridge also supports the creation of custom event buses.

EventBridge supports two types of `rules`:
- Event Pattern Rule - Developers define a pattern that is evaluated against the events flowing through the bus.
- Schedule Rule (cron) - An event is created based on a cron schedule.

When a rule is matched, the event is routed to one or more configurable `targets` (e.g., Lambda). EventBridge events can also be triggered based on a trigger schedule.
