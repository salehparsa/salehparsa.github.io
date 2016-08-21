---
layout: post
title: Allow/Deny Access to Certain IP Address in Tomcat 
categories: [tomcat, security, Linux, regex]
tags: [tomcat, security, regex]
fullview: true
comments: true
---

Tomcat is one of the most powerful webserver for implementing Java applications. As an administrator, you may need to restrict access to your webserver and practically you have got lots of options to achieve this requirement. However, One of the easiest ways to allow or deny access to Tomcat is via Remote Address filter of Tomcat _valves_ component. You can achieve it by adding following component to your _server.xml_ file:

~~~~~~~~
<Valve className="org.apache.catalina.valves.RemoteAddrValve" Attribute" />
~~~~~~~~

According to <a href="https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html"><b>Apache Tomcat documentation</b></a>, Remote Address has following attributes:

<ul>
  <li>allow</li>
  <li>deny</li>
  <li>denyStatus</li>
  <li>addConnectorPort</li>
  <li>invalidAuthenticationWhenDeny</li>
</ul>


<h5>Allow Certain IP Addresses:</h5> 

For example, if you want to allow IP _x.x.x.x_ , _y.y.y.y_ and subnet of _z.z.z.*_ you need to add following:

~~~~~~~~
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="x.x.x.x,y.y.y.y,z.z.z.*" />
~~~~~~~~

Above configuration forces Tomcat to allow clients with these specific IP addresses only.

<h5>Allow localhost to access via default port while other addresses are accessible via 1234:</h5>

~~~~~~~~
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
   addConnectorPort="true"
   allow="127\.\d+\.\d+\.\d+;\d*|::1;\d*|0:0:0:0:0:0:0:1;\d*|.*;1234"/>
~~~~~~~~

With above configuration, localhost _(127.0.0.1)_ is able to access tomcat via default connector port while all other users are accessing tomcat via port 1234.

In this example, _addConnectorPort_ configured as true and it means, tomcat compares allowed IP addresses against <b>IP:PORT</b> where IP is the client IP and PORT is the <b>Tomcat connector port</b>. 

<h4>Note:</h4>

<ul>
  <li>If you are using DHCP, this configuration may not be suitable for you since your IP Address may change via DHCP.</li>
  <li>If your application is integrated with other applications, you need to ensure that IP Address of those applications is listed in Tomcat. Otherwise, they are not able to communicate.</li>
  <li>You are able to add Regex instead of individually adding IP Addresses</li>
</ul>
