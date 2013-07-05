sandbox-ofsconnectorpoc
=======================
These projects are to provide example usage for invoking T24 Enterprise comoponents with TAFJ R12_201213 and TAFC PB201213

Navigate to the PROJECT_HOME (directory where you have loaded the Git repository) and setup build and run environment by executing the bat file (Please update the TAFJ_MB_DIR in the file according to your machine path)
<pre>
	$ dev
</pre>

## EB_OFSConnector

This actual T24 component to wrap OFS.BULK.MANAGER calls into component framework. Please make sure you have this component deplyed in your TAFJ or TAFC area. Following component artefacts can be found under 
$TAFJ_MB_DIR/T24/Component/OFSConnector/lib directory; 
<pre>
		1. Under data/java/t24-OFSConnectorService-Data.jar	--> Component Data Objects, Converter Objects and Response Objects, applicable for TAFC and TAFJ
		2. Under ejb
			2.1 tafc/t24-OFSConnectorService.ear	--> Component EJB interface for TAFC
			2.2 tafj/t24-OFSConnectorService-ejb.jar--> Component EJB interface for TAFJ
		3. Under proxyAdaptor/java/t24-OFSConnectorService-ProxyAdaptor.jar	--> Component Proxy Adaptor classes for external call out to 3rd party, applicable for TAFJ
		4. Under t24service/java/t24-OFSConnectorService.jar	--> Component Java API and jBC Implementation of Component service, applicable for TAFJ
		5. Under ws/java
			5.1 t24-OFSConnectorService-provider.jar	--> Component Provder API, applicable for TAFC and TAFJ (configurations needs to be changed to activate the related TAF impl)
			5.2 t24-OFSConnectorService-jws.aar	--> Component Axis2 deployable web service archive, applicable for TAFC and TAFJ (internally uses Component Provider API to call jBC Impl)
</pre>

# jBoss Security Config

As few of the following samples will be using secure component api's, you will be required to configured jBoss for the same to provide the security domain, users etc. 
Note: We will be using jboss default login modules to serve our need, but same can be achieved using industry standard LDAP server's, all you need to make sure that USER context and role is available.

Lets define a new jBoss Secutiry domain by adding following xml in 'JBOSS_HOME/server/&lt;profile&gt;/conf/login-config.xml' file;
<pre>
	<!-- T24 Application Login Module -->
	&lt;application-policy name="T24App"&gt;
		&lt;authentication&gt;
			&lt;login-module code="org.jboss.security.auth.spi.UsersRolesLoginModule" flag="required"&gt;
				&lt;module-option name="usersProperties"&gt;props/jbossws-users.properties&lt;/module-option&gt;
				&lt;module-option name="rolesProperties"&gt;props/jbossws-roles.properties&lt;/module-option&gt;
				&lt;module-option name="unauthenticatedIdentity"&gt;anonymous&lt;/module-option&gt;
			&lt;/login-module&gt;
		&lt;/authentication&gt;
	&lt;/application-policy&gt;
</pre>

Above configurations defines a new security domain in jboss as 'T24App' with user configuration file located at './props/jbossws-users.properties' and user rols configuration file located at './props/jbossws-roles.properties'

Now add a new user in jboss as 'SSOUSER' with password '123456' by adding following line in 'JBOSS_HOME/server/&lt;profile&gt;/conf/props/jbossws-users.properties';
<pre>
	SSOUSER=123456
</pre>

And define a role for the above user SSOUSER as 't24user' by adding the following line in 'JBOSS_HOME/server/&lt;profile&gt;/conf/props/jbossws-roles.properties';
<pre>
	SSOUSER=t24user
</pre>

NOTE: As you can see we have defined a jBoss user called SSOUSER, once authenticated 'SSOUSER' will be our PRINCIPAL and T24 needs to load the context for this user to execute the request. For that we need to define user
profile within T24 as SIGN.ON.NAME = SSOUSER (password can be antyhing for this user as it will be ignored during JF.VALIDATE.SIGN.ON) and one of the ATTRIBUTES of this user profile must be set to 'PREAUTHENTICATED'.

# TAFJ Samples

## Installing TAFJ Dependencies and Starting TAFJ DB
	
To compile and run the different samples we need to install few libraries from TAFJ into local maven repositories. This can be done by applying the following command;

NOTE: Please make sure TAFJ_HOME is set
<pre>
	mvn install:install-file -DgroupId=com.temenos -DartifactId=tafj-common -Dversion=dev -Dpackaging=jar -Dfile=%TAFJ_HOME%/lib/TAFJCommon.jar -DgeneratePom=true
	mvn install:install-file -DgroupId=com.temenos -DartifactId=tafj-core -Dversion=dev -Dpackaging=jar -Dfile=%TAFJ_HOME%/lib/TAFJCore.jar -DgeneratePom=true
	mvn install:install-file -DgroupId=com.temenos -DartifactId=tafj-locking -Dversion=dev -Dpackaging=jar -Dfile=%TAFJ_HOME%/lib/TAFJLocking.jar -DgeneratePom=true
	mvn install:install-file -DgroupId=com.temenos -DartifactId=tafj-collectorclient -Dversion=dev -Dpackaging=jar -Dfile=%TAFJ_HOME%/lib/T24CollectorClient.jar -DgeneratePom=true
	mvn install:install-file -DgroupId=com.temenos -DartifactId=tafc-jremote -Dversion=dev -Dpackaging=jar -Dfile=%TAFJ_HOME%/lib/jremote.jar -DgeneratePom=true
	mvn install:install-file -DgroupId=com.temenos -DartifactId=component-framework -Dversion=dev -Dpackaging=jar -Dfile=%TAFJ_HOME%/ext/tComponentFramework.jar -DgeneratePom=true
	mvn install:install-file -DgroupId=com.temenos -DartifactId=t24-precompiled -Dversion=dev -Dpackaging=jar -Dfile=%TAFJ_HOME%/../T24/lib/R12GA.jar -DgeneratePom=true
</pre>

Make sure Databse configured with TAFJ MB is up and running

## Installing OFSConnector Dependencies
<pre>
	// Component Data JAR
	mvn install:install-file -DgroupId=com.temenos -DartifactId=eb_ofsconnector-data -Dversion=dev -Dpackaging=jar -Dfile=%OFSCONNECTOR_LIB%/data/java/t24-OFSConnectorService-Data.jar -DgeneratePom=true
	
	// Component Java API JAR
	mvn install:install-file -DgroupId=com.temenos -DartifactId=eb_ofsconnector -Dversion=dev -Dpackaging=jar -Dfile=%OFSCONNECTOR_LIB%/t24service/java/t24-OFSConnectorService.jar -DgeneratePom=true
	
	// Component Provider API for TAFC
	mvn install:install-file -DgroupId=com.temenos -DartifactId=eb_ofsconnector-provider-tafc -Dversion=dev -Dpackaging=jar -Dfile=%OFSCONNECTOR_LIB%/ws/java/t24-OFSConnectorService-provider.jar -DgeneratePom=true
	
	// Component Provider API for TAFJ
	Note: Before applying following command, please upadte the spring configurations to load TAFJ provder. This can be done by updating the configurations in oFSConnectorServiceContext.xml in the following jar
	mvn install:install-file -DgroupId=com.temenos -DartifactId=eb_ofsconnector-provider-tafj -Dversion=dev -Dpackaging=jar -Dfile=%OFSCONNECTOR_LIB%/ws/java/t24-OFSConnectorService-provider.jar -DgeneratePom=true
	
	// Component EJB JAR For TAFJ
	mvn install:install-file -DgroupId=com.temenos -DartifactId=eb_ofsconnector-ejb-tafj -Dversion=dev -Dpackaging=jar -Dfile=%OFSCONNECTOR_LIB%/ejb/tafj/t24-OFSConnectorService-ejb.jar -DgeneratePom=true
	
	// Component EJB JAR For TAFC
	Note: By default on TAFC, EJB is wrapped into an EARm simply extract the JAR from an EAR into the same directory and then apply following command;
	mvn install:install-file -DgroupId=com.temenos -DartifactId=eb_ofsconnector-ejb-tafc -Dversion=dev -Dpackaging=jar -Dfile=%OFSCONNECTOR_LIB%/ejb/tafc/t24-OFSConnectorService-ejb.jar -DgeneratePom=true
</pre>

## Sample 1. ofsconn-standalone-test - [TAFJ Only]
This sample demonstrate how to directly invoke the Component jBC Impl via Component Java API in a standalone TAFJ environment from a Java test class.
Refer to the test classes under com.temenos.services.ofsconnector.tafj.standalone package for an examples. Call to jBC Impl will reach as follows;
<pre>
	TestClass --> Component Java API --> Component jBC Impl
</pre>

### To build and run the test apply following command;
<pre>
	$ cd PROJECT_HOME/ofsconn-standalone-test
	mvn clean install
</pre>	
	
## Sample 2. ofsconn-standalone-provider-test [TAFJ Only]
This sample demonstrate how to invoke the Component jBC Implementation via Provider API in standalone TAFJ environment from a Java test class
Refer to the test classes under com.temenos.services.ofsconnector.tafj.standalone package for an examples. Calls to jBC Impl will reach as follows;
<pre>
	TestClass --> Component Provider API --> Component Java API --> Component jBC Impl
</pre>

### To run the project apply following command;
<pre>
	$ cd PROJECT_HOME/ofsconn-standalone-provider-test
	mvn clean install
</pre>	

## Sample 3. ofsconn-appsrv-provider-test [TAFJ Only]
This sample demonstrate how a web servlet can instatiate and invoke Component Java Provider API. This requires deploying Component API into TAFJ environment running in Application Server mode. 
Note: Make sure component service is deployed within TAFJ environment

### To build the project by apply following command;
<pre>
	cd PROJECT_HOME/ofsconn-appsrv-provider-test
	mvn clean install
</pre>

Once build finsihed successfully, navigate to ofsconn-appsrv-provider-test/target and deploy the ofsconn.war into JBOSS_HOME/server/&lt;profile&gt;/deploy directory and access the following URL on your browser;
<pre>
	http://localhost:9090/ofsconn/foo?ofsRequest=ENQUIRY.SELECT,,INPUTT/123456,CURRENCY-LIST
</pre>

# TAFC Samples

## Sample 1. ofsconn-standalone-provider-tafc-test [For TAFC Only]
This sample demonstrate how we can invoke jBC Impl from standalone Java Program using Component default Provider API by loading TOCF T24 Resource Adaptor in a open source JENCKS framework which provides a JCA conatiner for Provider API to lookup a jca resource and connecto to T24 tafc_agent

### To build and run the test apply following command;
<pre>
	$ cd PROJECT_HOME/ofsconn-standalone-provider-tafc-test
	$ mvn clean install
</pre>

## Sample 2. ofsconn-standalone-provider-tafc-customCXManager-test [For TAFC Only]
This sample demonstrate how we can invoke jBC Impl from standalone Java Program using Component Provider API by providing custom T24 connection manager to connect to tafc_agent
Note: Make sure you have started the tafc_agent on T24 Application server

### To build and run the test apply following command;
<pre>
	$ cd PROJECT_HOME/ofsconn-standalone-provider-tafc-customCXManager-test
	$ mvn clean install
</pre>

# TAFC & TAFJ Samples [Applicable for Both]

## Sample 1. ofsconn-standalone-ejb-test [For TAFC and TAFJ]

This sample will demonstrate how we can invoke jBC Impl using Component secure EJB API deployed in Application Server from a standalone client. The same example can be used for both TAFC and TAFJ system by updating the JNDI-name lookup according to the system you want to connect, switch between TADC and TAFJ by updating the Confiurations.java. Example will lookup the default resource in jBoss Applocation Server as follows (update the JNDI if you have deployed with different name);
<pre>
	- For TAFJ 	-> OFSConnectorServiceBeanTAFJ/remote
		- Calls to jBC Impl will reach as follows;
			TestClass --> Component EJB API --> Component Java API --> Component jBC Impl
	- For TAFC  -> t24-OFSConnectorService/OFSConnectorServiceBeanTAFC/remote	
		- Calls to jBC Impl will reach as follows;
			TestClass --> Component EJB API --> Component Provider API --> Component jBC Impl
</pre>

To deploy the component ejb service in jBoss, simply drop the 't24-OFSConnectorService-ejb.jar' for TAFJ and  't24-OFSConnectorService.ear' for TAFC present in '%OFSCONNECTOR_LIB%/ejb/[tafj/tafc]' directory into 'JBOSS_HOME/server/&lt;profile&gt;/deploy' directory.
Note: By default service EJB are security enabled and expect 'T24App' security domain to be present at deployment. If its not created please create one beofore deploying the EJB service and define a user 'SSOUSER' with password '123456' and role as 't24user' within jboss for authentication. SSOUSER will be passed a SessionContext to and EJB which will be then used to set context within T24 before invoking the jBC Impl.

### To compile and run the test class 'OfsConnectorComponentTest' apply folliwng command;
<pre>
	$ cd PROJECT_HOME/ofsconn-standalone-ejb-test
	$ mvn clean install
</pre>

## Sample 2. cf-wsjavaclient [For TAFC and TAFJ]
 
This project creates the Java clients for a web service by invoking the 'JAVA_HOME/bin/wsimport' utility on a deployed webservice. To generate first we need to deploy the Web Service in axis2. This can be done by copying the file './dist/release/OFSConnector/lib/ws/java/t24-OFSConnectorService-jws.aar' into 'JBOSS_HOME/server/R12GA/deploy/axis2.war/WEB-INF/services' directory, one the deployment is finished you can verify your new web service on folllowing link by accessing its WSDL;
<pre>
	For TAFJ MB: http://localhost:9090/axis2/services/OFSConnectorServiceWS?wsdl
	For TAFC MB: http://localhost:8080/axis2/services/OFSConnectorServiceWS?wsdl
</pre>

### Build and run the cf-wsjavaclient
<pre>
	$ cd PROJECT_HOME/cf-wsjavaclient
	$ mvn clean install
</pre>

## Sample 3. ofsconn-appserv-ejb-test [For TAFC and TAFJ]

This sample demonstrate how we can invoke component jBC Impl from a web application deployed in an JavaEE Application Server via Component EJB API. The instructions here are to test with TAFC system, but same logic can be applied to TAFJ by updating the JNDI name to lookup. This example will also demostrate how we can pass the Web Secutiry Context from Web Container to an EJB container as all our Component EJB by default secutiry enabled.
the web archive is also configured to be secured and using a 'T24App' as a security domain. Please make sure you have this secutiry doamin available in your jBoss area with user profile as 'SSOUSER=123456' and user role
'SSOUSER=t24user'.
Note: Make sure you have started the tafc_agent on T24 Application server

### To build the example war apply following command;
<pre>
	$ cd PROJECT_HOME/ofsconn-appserv-ejb-test
	$ mvn clean install
</pre>

Once the build is finished, yoiu will get XXXX.war under ofsconn-appserv-ejb-test/target directory, simply copy this file under 'JBOSS_HOME/server/&lt;profile&gt;/deploy' directory and access the following URL;
<pre>
	http://localhost:9090/ofsconn-ejb/foo?ofsRequest=ENQUIRY.SELECT,,INPUTT/123456,CURRENCY-LIST
</pre>

As soon as you will try to access the above URL, application server will challenge you to provide a user name and password. Provide a jboss user you have defined earlier as 'SSOUSER=123456', once authenticated 'SSOUSER' will be
the user context for that session and will be received by the web application, which then internally pass that context to component EJB as Signle Sign On, which internally call T24 with the same context.
