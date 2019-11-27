---
title: Dispatcher in the Cloud
seo-title: Dispatcher in the Cloud
description: Dispatcher in the Cloud 
seo-description: Dispatcher in the Cloud 
---

# Dispatcher in the Cloud {#Dispatcher-in-the-cloud}

## Apache and Dispatcher configuration and testing {#apache-and-dispatcher-configuration-and-testing}

This section describes how to structure the AEM as a Cloud Service Apache and Dispatcher configuration, as well as how to validate  and run it locally before deploying to Cloud environments. It also describes debugging in Cloud environments. For additional information about Dispatcher, see the [AEM Dispatcher documentation](https://docs.adobe.com/content/help/en/experience-manager-dispatcher/using/dispatcher.html).

>[!NOTE]
>Windows users will need to use Windows 10 Professional or other distributions that support Docker. This is a pre-requisite for running and debugging Dispatcher on a local computer. The sections below include commands using the Mac or Linux versions of the SDK, but the Windows SDK can be used in a similar way.

## Dispatcher SDK {#dispatcher-sdk}

The Dispatcher SDK1 provides:

* A vanilla file structure containing the configuration files to include in a maven project for dispatcher;
* Tooling for customers to validate a dispatcher configuration locally;
* A Docker image that brings up the dispatcher locally.

For pre-release, the dispatcher SDK will be distributed via the pre-release website.

## Extracting the SDK {#extracting-the-sdk}

Download the shell script to a folder on your machine, make it executable and run it. It will extract the Dispatcher SDK files underneath the directory you stored it in.

## File Structure {#file-structure}

The structure of the project's dispatcher subfolder is described below and should be copied into the maven project dispatcher folder:

```
./
├── conf.d
│   ├── available_vhosts
│   │   └── default.vhost
│   ├── dispatcher_vhost.conf
│   ├── enabled_vhosts
│   │   ├── README
│   │   └── default.vhost -> ../available_vhosts/default.vhost
│   └── rewrites
│   │   ├── default_rewrite.rules
│   │   └── rewrite.rules
│   └── variables
|       ├── custom.vars
│       └── global.vars
└── conf.dispatcher.d
    ├── available_farms
    │   └── default.farm
    ├── cache
    │   ├── default_invalidate.any
    │   ├── default_rules.any
    │   └── rules.any
    ├── clientheaders
    │   ├── clientheaders.any
    │   └── default_clientheaders.any
    ├── dispatcher.any
    ├── enabled_farms
    │   ├── README
    │   └── default.farm -> ../available_farms/default.farm
    ├── filters
    │   ├── default_filters.any
    │   └── filters.any
    ├── renders
    │   └── default_renders.any
    └── virtualhosts
        ├── default_virtualhosts.any
        └── virtualhosts.any
```

Below is an explanation of notable files that can be modified:

**Customizable Files**

The following files are customizable and will get transferred to your Cloud instance on deployment:

* `conf.d/available_vhosts/<CUSTOMER_CHOICE>.vhost`

You can have one or more of these files. They contain `<VirtualHost>` entries that match host names and allow Apache to handle each domain traffic with different rules. Files are created in the `available_vhosts` directory and enabled with a symbolic link in the `enabled_vhosts` directory. From the `.vhost` files, other files like rewrites and variables will be included.

* `conf.d/rewrites/rewrite.rules`

This file is included from inside your `.vhost` files. It has a set of rewrite rules for `mod_rewrite`.

>[!NOTE]
>
>At this time, a single rewrite file must be used rather than site specific files. That file size must be less than 1MB.

* `conf.d/variables/custom.vars`

This file is included from inside your `.vhost` files. You can put defines for Apache variables in this location.

* `conf.d/variables/global.vars`

This file is included from inside the `dispatcher_vhost.conf` file. You can change your dispatcher and rewrite log level in this file.

* `conf.dispatcher.d/available_farms/<CUSTOMER_CHOICE>.farm`

You can have one or more of these files, and they contain farms to match host names and allow the dispatcher module to handle each farm with different rules. Files are created in the `available_farms` directory and enabled with a symbolic link in the `enabled_farms` directory. From the `.farm` files, other files like filters, cache rules and others will be included.

* `conf.dispatcher.d/cache/rules.any`

This file is included from inside your `.farm` files. It specifies caching preferences.

* `conf.dispatcher.d/clientheaders/clientheaders.any`

This file is included from inside your `.farm` files. It specifies what request headers should be forwarded to the backend.

* `conf.dispatcher.d/filters/filters.any`

This file is included from inside your `.farm` files. It has a set of rules that change what traffic should be filtered out and not make it to the backend.

* `conf.dispatcher.d/virtualhosts/virtualhosts.any`

This file is included from inside your `.farm` files. It has a list of host names or URI paths to be matched by glob matching. This determines what backend to use to serve a request.

The above files reference the immutable configuration files listed below. Changes to the immutable files will not be processed by dispatchers in Cloud environments.

**Immutable Configuration Files**

These files are part of the base framework and enforce standards and best practices. The files are considered immutable because modifying or deleting them locally will have no impact on your deployment, as they will not get transferred to your Cloud instance.

* `conf.d/available_vhosts/default.vhost`

Contains a sample virtual host. For your own virtual host, create a copy of this file, customize it, go to `conf.d/enabled_vhosts` and create a symbolic link to your customized copy.

* `conf.d/dispatcher_vhost.conf`

Part of the base framework, used to illustrate how your virtual hosts and global variables are included.

* `conf.d/rewrites/default_rewrite.rules`

Default rewrite rules suitable for a standard project. If you need customization, modify `rewrite.rules`. In your customization, you can still include the default rules first, if they suit your needs.

* `conf.dispatcher.d/available_farms/default.farm`

Contains a sample dispatcher farm. For your own farm, create a copy of this file, customize it, go to `conf.d/enabled_farms` and create a symbolic link to your customized copy.

* `conf.dispatcher.d/cache/default_invalidate.any`

Part of the base framework, gets generated on startup. You are **required** to include this file in every farm you define, in the `cache/allowedClients` section.

* `conf.dispatcher.d/cache/default_rules.any`

Default cache rules suitable for a standard project. If you need customization, modify `conf.dispatcher.d/cache/rules.any`. In your customization, you can still include the default rules first, if they suit your needs.

* `conf.dispatcher.d/clientheaders/default_clientheaders.any`

Default request headers to forward to the backend, suitable for a standard project. If you need customization, modify `clientheaders.any`. In your customization, you can still include the default request headers first, if they suit your needs.

* `conf.dispatcher.d/dispatcher.any`

Part of base framework, used to illustrate how your dispatcher farms are included.

* `conf.dispatcher.d/filters/default_filters.any`

Default filters suitable for a standard project. If you need customization, modify `filters.any`. In your customization, you can still include the default filters first, if they suit your needs.

* `conf.dispatcher.d/renders/default_renders.any`

Part of base framework, this file gets generated on startup. You are **required** to include this file in every farm you define, in the `renders` section.

* `conf.dispatcher.d/virtualhosts/default_virtualhosts.any`

Default host globbing suitable for a standard project. If you need customization, modify `virtualhosts.any`. In your customization, you shouldn't include the default host globbing, as it matches **every** incoming request.

>[!NOTE]The AEM as a Cloud Service maven archetype will generate the same dispatcher configuration file structure.

The sections below describe how to validate the configuration locally so it can pass the associated quality gate in Cloud Manager when deploying an internal release.

## Local validation of Dispatcher configuration {#local-validation-of-dispatcher-configuration}

The validation tool is available in the SDK as a Mac OS, Linux, or Windows binary, allowing customers to run the same validation that Cloud Manager will perform while building and deploying a release.

It is invoked as: `validator full [-d folder] [-w whitelist] zip-file`

The tool validates the Apache and dispatcher configuration contained in the zip file. It scans all files with pattern `conf.d/enabled_vhosts/*.vhost` and checks that only whitelisted directives are used. The directives allowed in Apache configuration files can be listed by running the validator's whitelist command:

```

$ validator whitelist
Cloud manager validator 2.0.4
 
Whitelisted directives:
  <Directory>
  ...
  
  ```

The whitelist contains a list of Apache directives that are expected in the customer configuration. If a directive is not whitelisted, the tool logs an error and returns a non-zero exit code. If no whitelist is given on the command line, the tool uses a default whitelist, which is what Cloud Manager will use for validation before deploying to Cloud environments.

 Also, it further scans all files with pattern `conf.dispatcher.d/enabled_farms/*.farm` and checks that:

* No filter rule exists that uses allows via `/glob` (see [CVE-2016-0957](https://nvd.nist.gov/vuln/detail/CVE-2016-0957) for more details)
* No admin feature is exposed. For example, access to paths such as `/crx/de or /system/console`. 

When run against your maven artifact or your `dispatcher/src` subdirectory, it will report validation failures:

 ```

$ validator full dispatcher/src
Cloud manager validator 1.0.4
2019/06/19 15:41:37 Apache configuration uses non-whitelisted directives:
  conf.d/enabled_vhosts/aem_publish.vhost:46: LogLevel
2019/06/19 15:41:37 Dispatcher configuration validation failed:
  conf.dispatcher.d/enabled_farms/999_ams_publish_farm.any: filter allows access to CRXDE

```

Note that the validation tool reports only the prohibited use of Apache directives that have not been whitelisted. It does not report syntactical or semantical problems with your Apache configuration, as this information is only available to Apache modules in a running environment.

When no validation failures are reported, your configuration is ready for deployment.

## Testing your Apache and Dispatcher configuration locally {#testing-apache-and-dispatcher-configuration-locally}

It is also possible to test drive your Apache and Dispatcher configuration locally. It requires Docker to be installed locally and your configuration to pass the validation as described above.

By using the "`-d`" parameter, the validator outputs a folder with all the configuration files needed by the dispatcher.

Then, the `docker_run.sh` script can point to that folder, starting the container with your configuration.

```

$ validator full -d out src/dispatcher
2019/06/19 16:02:55 No issues found
$ docker_run.sh out docker.for.mac.localhost:4503 8080
Running script /docker_entrypoint.d/10-create-docroots.sh
Running script /docker_entrypoint.d/20-wait-for-backend.sh
Waiting until aemhost is available
aemhost resolves to xx.xx.xx.xx
Running script /docker_entrypoint.d/30-allowed-clients.sh
Starting httpd server
...

```

This will start the dispatcher in a container with its backend pointing to an AEM instance running on your local Mac OS machine at port 4503.

## Debugging your Apache and Dispatcher configuration {#debugging-apache-and-dispatcher-configuration}

The following strategy can be used to increase the log output for the dispatcher module and see the result of the `RewriteRule` evaluation in both local and cloud environments.

Log levels for those modules are defined by the variables `DISP_LOG_LEVEL` and `REWRITE_LOG_LEVEL`. They can be set in the file `conf.d/variables/global.vars`. Its relevant part follows:

```

# Log level for the dispatcher
#
# Possible values are: Error, Warn, Info, Debug and Trace1
# Default value: Warn
#
# Define DISP_LOG_LEVEL Warn
 
# Log level for mod_rewrite
#
# Possible values are: Error, Warn, Info, Debug and Trace1 - Trace8
# Default value: Warn
#
# To debug your RewriteRules, it is recommended to raise your log
# level to Trace2.
#
# More information can be found at:
# https://httpd.apache.org/docs/current/mod/mod_rewrite.html#logging
#
# Define REWRITE_LOG_LEVEL Warn

```

When running the Dispatcher locally, logs are also directly printed to the terminal output.

## Different Dispatcher configurations per environment {#different-dispatcher-configurations-per-environment}

At this time, the same dispatcher configuration is applied to all AEM as a Cloud Service environments. The runtime will have an environment variable `ENVIRONMENT_TYPE` that contains the current run mode (dev, stage or prod) as well as a define. The define can be `ENVIRONMENT_DEV`, `ENVIRONMENT_STAGE` or `ENVIRONMENT_PROD`. In the Apache configuration, the variable can be used directly in an expression. Alternatively, the define can be used to build logic:

```

# Simple usage of the environment variable
ServerName ${ENVIRONMENT_TYPE}.company.com
 
# When more logic is required
<IfDefine ENVIRONMENT_STAGE>
  # These statements are for stage
  Define VIRTUALHOST stage.example.com
</IfDefine>
<IfDefine ENVIRONMENT_PROD>
  # These statements are for production
  Define VIRTUALHOST prod.example.com
</IfDefine>

```

In the Dispatcher configuration, the same environment variable is available. If more logic is required, define the variables as shown in the example above and then use them in the Dispatcher configuration section:

```

/virtualhosts {
  { "${VIRTUALHOST}" }
}

```

When testing your configuration locally, you can simulate different environment types by passing the variable `DISP_RUN_MODE` to the `docker_run.sh` script directly:

```

$ DISP_RUN_MODE=stage docker_run.sh out docker.for.mac.localhost:4503 8080

```

For a complete list of options and variables available, run the script `docker_run.sh` without arguments.

## Viewing the Dispatcher configuration in use by your Docker container {#viewing-dispatcher-configuration-in-use-by-docker-container}

With environment specific configurations, it can be difficult to determine what the actual Dispatcher configuration looks like. After having started your docker container with `docker_run.sh` it can be dumped as follows:

* Determine the docker container ID in use:

```

$ docker ps
CONTAINER ID       IMAGE
d75fbd23b29        adobe/aem-ethos/dispatcher-publish:...

```

*  Execute the following command line with that container ID:

```

$ docker exec d75fbd23b29 httpd-test
# Dispatcher configuration: (/etc/httpd/conf.dispatcher.d/dispatcher.any)
/farms {
  /publishfarm {
    /clientheaders {
...

```

## Main Differences between AMS Dispatcher and AEM as a Cloud Service {#main-differences-between-ams-dispatcher-configuration-and-aem-as-a-cloud-service}

As described on the reference page above, the Apache and Dispatcher configuration in AEM as a Cloud Service is quite similar to the AMS one. The main differences are:

* In AEM as a Cloud Service, some Apache directives may not be used (for example `Listen` or `LogLevel`)
* In AEM as a Cloud Service, only some pieces of the Dispatcher configuration can be put in include files and their naming is important. For example, filter rules that you want to reuse across different hosts must be put in a file called `filters/filters.any`. See the reference page for more information.
* In AEM as a Cloud Service there is extra validation to disallow filter rules written using `/glob` to prevent security issues. Since `deny *` will be used rather than `allow *` (which cannot be used), customers will benefit from running the Dispatcher locally and doing trial and error, looking at the logs to know exactly what paths the Dispatcher filters are blocking in order for those can be added.

## Dispatcher and CDN {#dispatcher-cdn}

The data flow is as follows:

1. URL put in browser
2. Request made to CDN mapped in DNS to that domain
3. If content is fully cached on CDN, CDN serves it to the browser
4. If content is not fully cached, the CDN calls out (reverse proxy) to the dispatcher
5. If content is fully cached on dispatcher, dispatcher serves it to the CDN
6. If content is not fully cached, the dispatcher calls out (reverse proxy) to the AEM publish
7. Content rendered by browser

### CDN {#cdn}

AEM offers two options:

1. Managed CDN - AEM's out-of-the-box CDN.
2. Unmanaged CDN - Customer brings their own CDN and is entirely responsible for managing it.

The first option is highly recommended and Adobe is not responsible for the result of any misconfiguration when using the second option.

#### Managed CDN  {#managed-cdn}

After Adobe provisions the CDN, you should create a CNAME, mapping your application's domains to an Adobe controlled domain hosted at the managed CDN domain.

The CDN acts as a layer 7 web application firewall, which requires SSL termination, hence it will need a customer-signed SSL certificate. During the pre-release phase, you have to provide the certificate to Adobe through manual processes.

At the time of GA, you should upload the certificate to Cloud Manager, which will in turn upload it to the CDN. When SSL certificates are expiring, you will be notified so you can refresh the certificates in Cloud Manager.

#### Unmanaged CDN {#unmanaged-cdn}

You may manage your own CDN, provided:

1. You have an existing CDN. 
2. You will manage it.
3. Your application does not make extensive API calls to the CDN that are not already accommodated in AEM architecture.
4. AEM as a Cloud Service is able to establish that the end-to-end system functions properly.
5. You will need to provide Adobe with the whitelist of CDN urls to allow.

Adobe will provide a AEM Cloud url for to use as your CDN's origin. 


#### CDN cache invalidation {#CDN-cache-invalidation}

Cache invalidation follows these rules:

* In general, HTML content is cached in the CDN for 5 minutes, based on the cache-control header emitted by the dispatcher.
* Client libraries (JavaScript and CSS) are cached indefinitely using the cache-control set to either immutable or 30 days for older browsers which don't respect the immutable value. Note that the client libraries are served on a unique path that changes if the client libraries change. In other words, HTML that references the client libraries will be produced as needed so you can experience new content as it is published.
* By default, images are not cached.

## Explicit dispatcher cache invalidation {#explicit-invalidation}

Prior to AEM as a Cloud Service, there were 2 ways of invalidating the dispatcher cache.

1. Invoke the replication API, specifying the publish dispatcher flush agent
2. Directly calling the `invalidate.cache` API (e.g. POST /dispatcher/invalidate.cache)

The `invalidate.cache` approach will no longer be supported since it addresses only a specific dispatcher node.
AEM as a Cloud Service operates at the service level, not the individual node level and thus the invalidation instructions in the [Dispatcher Help](https://docs.adobe.com/content/help/en/experience-manager-dispatcher/using/dispatcher.html) documentation are no longer accurate. 
Instead, the replication API approach should be used. [API documentation](https://helpx.adobe.com/experience-manager/6-5/sites/developing/using/reference-materials/javadoc/com/day/cq/replication/Replicator.html) is available and for an example of flushing the cache, see the [API example page](https://helpx.adobe.com/experience-manager/using/aem64_replication_api.html) and specifically the CustomStep example issuing a replication action of type ACTIVATE to all available agents. 

The diagram below illustrates this.

<!-- [CDN](assets/cdn.png "CDN") -->

<!-- See [Apache and Dispatcher Configuration and Testing](../developing/introduction/developer-experience.md#apache-and-dispatcher-configuration-and-testing) for instructions on how a developer can configure apache and the dispatcher module. -->

### Dispatcher Cache Invalidation during Activation/Deactivation {#cache-activation-deactivation}

This publish-triggered invalidation is the same as quickstart:
When the publish instance receives a new version of a page or asset from the author (via the replication and pipeline queue), it uses the flush agent to invalidate appropriate paths on its dispatcher. The updated path is removed from the dispatcher cache, together with its parents, up to some level that you can configure with the [statfileslevel](https://docs.adobe.com/content/help/en/experience-manager-dispatcher/using/configuring/dispatcher-configuration.html#invalidating-files-by-folder-level).

### Content Freshness and Version Consistency {#content-consistency}

* Pages are made of HTML, Javascript, CSS, and images.
* You are encouraged to leverage the clientlibs framework to import Javascript and CSS resources into HTML pages, taking into account dependencies between JS libraries.
* Automatic version management is provided, meaning that developers can check in changes to JS libraries in source control, and the latest version will be made available when a release is pushed. Without this, developers would need to manually change HTML with references to the new version of the library, which is especially onerous if many HTML templates share the same library.
* When the new versions of libraries are released to production, the referencing HTML pages are updated with new links to those updated library versions. Once the browser cache has expired for a given HTML page, there is no concern that the old libraries will be loaded from the browser cache since the refreshed page (from AEM) now is guaranteed to reference the new versions of the libraries. In other words, a refreshed HTML page will include all the latest library versions.