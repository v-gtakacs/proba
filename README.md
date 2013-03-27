Description
===========

Installs and configures Apache Tomcat.

Requirements
============

Platform:

* CentOS 6+, Red Hat 6+, Fedora (OpenJDK, Oracle)

The following cookbooks are dependencies:

* s3curl
* java

Attributes
==========

* `node["tomcat"]["instance_name"]` - This attribute has two purposes. Currently it is used for the HTTPS setup to specify where the keystore and truststore password is defined within the `credentials` encrypted data bag item. Later, when multiple Tomcat instances will be supported, this attribute will define the currenty deployed instance name.  **Default value:** `'tomcat'`  
**See also:** [Setup The HTTPS Connector.](#setup-the-https-connector)
* `node["tomcat"]["shutdown_port"]` - The shutdown port number of the Tomcat instance. **Default value:** `8005`
* `node["tomcat"]["http_port"]` - The HTTP port number of the Tomcat instance. **Default value:** `8080`
* `node["tomcat"]["https_port"]` - The HTTPS port number of the Tomcat instance. **Default value:** `8443`  
**See also:** [Setup The HTTPS Connector.](#setup-the-https-connector)  
* `node["tomcat"]["ssl_enabled"]` - This attribute indicates whether the Tomcat instance provides a HTTPS connector too. **Default value:** `false`  
**See also:** [Setup The HTTPS Connector.](#setup-the-https-connector)
* `node["tomcat"]["unpack_wars"]` - This attribute indicates whether the Tomcat instance unpacks the deployed war file(s) or not. **Default value:** `true`

Data Bags
=========

* `credentials` - Encrypted data bag defines sensitive parameters: s3reader name, keystore file password and truststore file password (if SSL is enabled).

Setup The HTTPS Connector
=========================

* Enable the SSL connector by overriding the `node["tomcat"]["ssl_enabled"]` attribute (for example at role level) with value `true`.
* Override the HTTPS port number if needed, by overriding the `node["tomcat"]["https_port"]` attribute (for example at role level).
* Override the instance name too, by overriding the `node["tomcat"]["instance_name"]` attribute (for example at role level). This is usually the name of your service.
* Create a new element within the `credentials` encrypted data bag item with the same name defined with the `node["tomcat"]["instance_name"]` attribute. This element must contain two key-value pairs: `truststore_file_password` and `keystore_file_password`. These define the passwords for the jks files included within the generated server.xml. You need to define these settings for each environment.  
**For example:** `"instance_name": { "truststore_file_password": "password1", "keystore_file_password": "password2" }`
* The truststore and keystore files must be placed into the `/opt/tomcat/conf` directory during the deployment by **your own cookbook** (the coookbook which refers to the `tomcat` cookbook). So you need to implement the followings **within your own cookbook**:
   * Create the jks files using the following naming convention and move these files into the `files/default` directory within your own cookbook:
      * The name of the truststore file is `trustedca.jks`.
      * The name of the keystore file is `<environment_name>.jks` (for example: ewetest.jks).
   * Add the following operations to your Chef recipe within your own cookbook:

```ruby
cookbook_file "/opt/tomcat/conf/#{node.chef_environment}.jks" do
  source "#{node.chef_environment}.jks"
  owner "webmaster"
  mode 00644
  notifies :restart, resources(:service => "tomcat")
end
cookbook_file "/opt/tomcat/conf/trustedca.jks" do
  source "trustedca.jks"
  owner "webmaster"
  mode 00644
  notifies :restart, resources(:service => "tomcat")
end
```

