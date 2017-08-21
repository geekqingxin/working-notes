
# 1. Tomcat发布应用

[官方](http://tomcat.apache.org/tomcat-8.5-doc/deployer-howto.html#Introduction)

## 1.1 Startup Deployment of Tomcat

[Tomcat Deploy Procedures](https://www.mulesoft.com/tcat/tomcat-deploy-procedures)

Also knowns as folder deployment, startup deployment is the easiest way to deploying a web application to your server.

To have Tomcat deploy an application automatically when it starts, simple copy either an uncompressed web application (also known as an "exploded" application) or a compressed application, or WAR, to the appBase folder specified in your Host configuration.

When Tomcat starts, it will deploy all applications in the appBase, in the following order:

* Applications defined by Context elements in either "$CATALINA_BASE/conf/server.xml" (discouraged), "$CATALINA_BASE/conf/[EngineName]/[HostName]/[ApplicationName].xml", "or $CATALINA_BASE/webapps/[ApplicationName]/META-INF/context.xml"
* Uncompressed (exploded) applications located in the appBase without Context Descriptors.
* docBase applications that are compressed in WAR files, along with WARs associated with an existing application, which will be used to automatically redeploy the application.