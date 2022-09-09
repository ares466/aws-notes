# OpsWorks

OpsWorks is configuration management service that provides an AWS-managed version of Chef and Puppet. 

Chef and Puppet are tools that allow you to write code to manage server configuration.

**Puppet** allows you to define the desired state and Puppet will make it happen. With Puppet, you do not have a lot of control on how the updates are accomplished.

In contrast, **Chef** allows you to script out the updates in Ruby.

AWS OpsWorks can be run in one of three modes:
- **Puppet Enterprise**: AWS-manged Puppet Master Server
- **Chef Automate**: AWS-managed Chef servers
- **OpsWorks**: Serverless AWS-integrated Chef

> [Exam Tip]
>
> If a question mentions recipes, cookbooks, or manifests, then the answer is likely related to OpsWorks.

OpsWorks consists of stacks, layers, recipes, and cookbooks.
- Stacks: Container of resources
- Layers: Specific function within a stack (e.g., web)

OpsWorks operates using lifecycles including setup, configure, deploy, undeploy, and shutdown.

OpsWorks runs on several instance types:
- 24/7: Always running
- Time-based: Runs during the specified time
- Load-based: Runs when needed, based on load

![OpsWorks](./static/images/opsworks.png)