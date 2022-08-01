# Instance Stores

Instance store volumes provide block storage devices to EC2.

Unlike EBS, instance stores are physically connected to only one EC2 host. Because they're physically connected, they offer the highest performance block storage in AWS.

Instance stores are included in the price of the EC2 instance.

Instance stores can only be connected at launch. They cannot be added afterward.

Instance stores are ephemeral. Any data on an instance store will be losts if the instance is moved, resized, or there is a hardware failure.