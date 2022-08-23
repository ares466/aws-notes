# SSM Parameter Store

SSM Parameter Store is designed to store configuration and secrets for applications.

Parameters can be stored as `String`, `StringList`, or `SecureString`. Parameters of type *SecureString* are encrypted using KMS keys.

SSM Parameter Store supports `versioning` for parameters.

SSM Parameter Store supports `hierarchical parameter storage`. Permissions can be delegated to specific parameters, or a hierarchy of parameters.

*Caption (below): Parameters can be stored in a hierarchy. Permissions can be granted for the hierarchy (e.g., /wordpress).*
```
/wordpress/
  dbuser (/wordpress/dbuser)
  dbpassword (/wordpress/dbpassword)
```

SSM Parameter Store publishes some `public parameters` from AWS, such as the latest AMI per region.
