+++
title = "Standing Up New Cxs"
description = "Recipe Summary"
categories = ["recipes"]
tags = ["[foo]"]
date = 2018-10-26T09:55:14-05:00
+++


### API Development Steps

 * Creating of new CXS 3.5 project
 * Create REST controller class
 * Create Process class and migrate validation logic from 3.0 to 3.5 Process class
 * For each FAST, create a service class (with hystrix setup) and migrate FAST Transaction
 * For each DB call, create a service class (with hystrix setup) and migrate DB native call to JPA call
 * For each CXS REST call, create a service class (with hystrix setup) and migrate to CXS REST call. If the provider is 3.5 REST app, use discovery client(ex. CFMS)
 * For each SOAP call, create a service class (with hystrix setup) and migrate SOAP using new Spring SOAP library in CALC 3.5
 * For each JMS, create a service class (with hystrix setup) and migrate JMS publish/subscribe using Spring JMS in CXS 3.5
 * Additional items:
  	* Integrate FCL/ADMC authentication if needed.
  	* Migrate any properties to yml files. Make sure reloadable entries are configured correctly.
  	* Migrate Resource Bundles using Spring Resource Bundles.
  	* Modify error codes as per 3.0 mapping file
 * Resolve any critical/major SonarQube issues
 * Validate overall integration of all 3.5 sub components by verifying functional sanity of the end point.

### Application Setup

#### Pre-requisites

##### Required Software's
  * Java Development Kit					-		1.8												
  * IDE 									-		Eclipse
  * REST Controller Framework				-		Spring Web MVC
  * Micro Service Application Framework 	-		Spring Boot
  * Logging									-		Spring logback
  * Version Control System					-		GIT
  * Build Tool								-		Maven
  * Code Coverage 							-		JaCoCo
  * Static Analysis							-		SonarQube
  * Database								-		Oracle
  * Unit Testing							-		JUnit
  * BDD Testing								-		Cucumber
  * Deployment and Release					-		Enterprise Jenkins (Cloud Bees)
  * Spring Cloud Config (Server and Cloud)	-		Spring Cloud
  * Spring Batch for Model and Transformation-		Spring Boot Starter Data JPA
  * Hystrix Dashboard Aggregator 				-	Spring Cloud Turbine (Linux/64-bit)
  * Secret Key Config Manager					-	Vault
  * Service Registration and Discovery			-	Eureka
  * Load Balancer (Client SLB) 					-	Netflix Ribbon
  * JMS											-	Spring JMS
  * Type-safe Bean Mapping						-	Mapstruct
  * Testing										-	Postman


##### Access
Follow below steps to get access for GIT Repo

 * Go to [Image] (https://oam.secure.fedex.com/oam/server/obrareq.cgi?encquery%3DG7YDZHSEb%2BcFEN6pKfTtapI7Cj%2FrFddmeF4EsDnhgoDIbsbCVzwSDczpEDqyRzaN9R6z2SsjSxm168ehIUxcz9pPF15m7YMjHxBtoOUE2Xu4SuvQ1JgufXKCVunpFtqGIQe3nNjNRTBEUFQ%2BL1%2Fm7OPeJgxNnsPNP5JDFx0h4sRIgiZwtqssbiY21riRymJITq2HA5xsNYlXMy3IzSYgHgTWnpHZY8LI1hHKG8uXTNMUymrNwrPjXmcRklRFs8MMpGqQqXomVKtQRny8pBhZmFx6a6cVkBOCBLTBb%2Fu%2F1ibTRS5fsF9%2BYSULTkHZyCby%20agentid%3DWTCApacheWebgate%20ver%3D1%20crmethod%3D2%26cksum%3D93168ca730cb2b390cbbc091eaf89ecf048de98e) in [FedEx home](http://home.fedex.com)
 * Go to Manager User Access
 * Enter Employee ID in Select Users tab
 * Enter EAI number in Select Access tab
 * Enter comments in Review and Submit tab and click Submit

### Development
#### Creating new CXS 3.5 Application

Refer [CXS 3.5 Maven Archetype guide](https://collab.purplehub.fedex.com/Communities/Customer%20Experience%20Service/Documents/CXS%203.5/CXS%203.5%20Maven%20Archetype%20Developers%20Guide_v3.5.docx) to create a new CXS 3.5 Micro Service

#### Integration with CXS 3.5 Supporting apps
##### Config Server
CXS connects to Config Service to fetch configurations from Config git yml files. Each application has common config entries, level specific entries.

Note: It is not recommended to have application dev yml file in Config Service as it intended only in dev mode and doesn’t need integration.

Following  steps explain how to setup Config Service in local and connect.

 * [Config Service](https://conexus.prod.fedex.com:9443/git/3531288/fedex_com_cxsconfigservice.git)
 * Run the Config Service application ConfigServerApplication. By default config service spring boot app [starts](http://localhost:8888/config) on the port 8888.
 * Configurations for level specific can be fetched using http://localhost:8888/config/{cxs-application-name}/{profile}
 * CXS connects to config server based on configuration defined in bootstrap-dev.yml. If fail fast is marked as false, CXS will ignore even if config server is not connecting.
 e.g.,

 ```
 spring:
    cloud:
        config:
            fail-fast: false
            retry:
                initial-interval: 1000
                max-interval: 2000
                max-attempts: 100
            uri: http://localhost:8888/config

 ```

Following is a snippet for availabilitycxs from config server in local.
URL: http://localhost:8888/config/availabilitycxs/dev

```
{"name":"availabilitycxs","profiles":["dev"],"label":"master","version":"2116b1b2e920443a1e32266ed2df09de34504b93","state":null,"propertySources":[{"name":"https://conexus.prod.fedex.com:9443/git/3531288/cxs_devtest_configs.git/application-dev.yml","source":{"discovery.registry.user":"admin","discovery.registry.password":"admin","eureka.client.serviceUrl.defaultZone":"http://localhost:8761/regs/eureka/"}},{"name":"https://conexus.prod.fedex.com:9443/git/3531288/cxs_devtest_configs.git/application.yml","source":{"configserver.name":"fedex.cxs.configserver config server","configserver.status":"Connected to the fedex.cxs.configserver git config server!","spring.cloud.loadbalancer.retry.enabled":true,"cfms.ribbon.ConnectTimeout":1000,"cfms.ribbon.ReadTimeout":3000,"cfms.ribbon.MaxAutoRetries":0,"cfms.ribbon.MaxAutoRetriesNextServer":4,"cfms.ribbon.OkToRetryOnAllOperations":true,"eureka.server.waitTimeInMsWhenSyncEmpty":0,"eureka.server.enableSelfPreservation":false,"eureka.instance.prefer-ip-address":false,"eureka.instance.lease-renewal-interval-in-seconds":5,"eureka.instance.lease-expiration-duration-in-seconds":10,"eureka.instance.status-page-url-path":"${management.context-path}/info","eureka.instance.health-check-url-path":"${management.context-path}/health","eureka.instance.metadata-map.profile":"${spring.profiles.active}","eureka.instance.metadata-map.version":"${info.project.version}","eureka.client.enabled":true,"eureka.client.healthcheck.enabled":true,"eureka.client.instance-info-replication-interval-seconds":10,"eureka.client.registry-fetch-interval-seconds":10,"eureka.client.serviceUrl.defaultZone":"http://${discovery.registry.user:admin}:${discovery.registry.password:admin}@${discovery.registry.host:localhost}:${discovery.registry.port}/regs/eureka/","management.context-path":"/management","management.security.enabled":false,"management.health.jms.enabled":false,"management.health.config.enabled":false,"endpoints.health.sensitive":false}}]}
```

##### Registry/Eureka Service
CXS connects to Registry service which provides registration and lookup features in cloud environment.

*Note*: It is not required to setup local registry service to Develop CXS. CXS needs registry service if it has to use Discovery client for CFMS. In order to disable connecting to registry service, comment out the @EnableDiscoveryClient on the main application class.

Steps to setup Eureka service in Local:

 * Check out [Registry CXS](https://conexus.prod.fedex.com:9443/git/3531287/fedex_com_cxsregistryservice.git)
 * Build and run the application by running the main class - CxsregistryApplication.
 * Verify application [start up](http://localhost:8761/regs/registry)
 * CXS connects to registry based on below configuration in dev yml files.

```
#application-dev.yml
discovery:
  registry:
    user: admin
      password: admin
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/regs/eureka/
```
##### CFMS
CXS connects to CFMS to populate messages. CFMS can be invoked through over APIG or CXS directly. In CXS 3.5, CFMS is connected through Discovery Client capability of Spring Cloud. To configure the same, following entries need to be present in respective cxs yml files.

```
cfms.v3.service.hostname: cfms
cfms.v3.callThroughAPIG: false
cfms.v3.service.uri: /customermessage/v1/messages/info
cfms.v3.service.secured: false
cfms.v3.service.media.types: application/json
cfms.v3.service.response.version: apig-cxs-v3
cfms.v3.service.request.media.types: application/json
```

##### Hystrix Dashboard
CXS connects to Hystrix(through registry service) and publishes its metrics.
Follow below steps to setup HDBS in local view local CXS metrics.

 * Check out [HDBS](https://conexus.prod.fedex.com:9443/git/3531289/fedex_com_cxshystrixdashboard.git)
 * Build and run the application at HystrixMonitorApplication.
 * View the application at below URL's:
 	* http://localhost:7999/hdbs/hystrix.stream
 	* http://localhost:7999/hdbs/hystrix/monitor?stream=http%3A%2F%2Flocalhost%3A7999%2Fhdbs%2Fturbine.stream

##### DevFramework
Devframework integration is needed to make a secure call to any secured backend system. We need to configure the following:

 * Uncomment the devframework dependencies "css jar" pom entries in master pom xml present at the root of the respective project.
 * Path to client.properties file with keystore file location and other info. Usually there are only 2 set of files, one for lower levels (client.test.properties) and another one for PROD (client.prod.properties). Copy these files over from existing 3.0 application server to build-distribution/appsec directory in 3.5 project. These files are copied over to appsec directory on server during staging process. The configuration is done in resources/config/application-<profile>.yml as shown below
 * Keystore file (\*.p12). Usually there are only 2 set of files, one for lower levels (APP\*.test.p12) and another one for PROD (APP*.prod.p12). Copy latest files over from existing 3.0 application server to build-distribution/appsec directory in 3.5 project. These files are copied over to appsec directory on server during staging process. These files are referred to from client.properties file mentioned above
 * Path to security.properties file. Usually there are one such file per level (security.L0.properties, security.L1.properties etc. Copy these files over from existing 3.0 project or application server to resources directory in 3.5 project. These files are packaged in the main jar file for the application. The configuration is done in resources/config/application-<profile>.yml.
 * fp.properties file, copy this file from existing 3.0 application to resources directory in 3.5 project. This file is packaged in the main jar file for the application.
 * CDS uri and default service AppId. This is done in resources/config/application-<profile>.yml as shown below.

```
cxs:
    security:
        devFramework:
            clientPropertiesPath:  /opt/fedex/cfmscxs/appsec/client.test.properties
            securityPropertiesPath: security.L3.properties
            cds:
                defaultUri: http://cds-level3.ute.fedex.com/CommonDataService/clear/
                defaultServiceAppId: 943415_cds
```
 * Bootstrap devframework through a set of spring beans. Refer to SecurityConfiguration class in CFMS project(com.fedex.cxs.cfms.config.SecurityConfiguration.java) for the reference:

```
@Bean
    @DependsOn({"cdsSecurityAction","cdsSecurityRole","cdsSecurityAppRole",
    	"cdsSecurityGroupRole","cdsSecurityUserRole","cdsSecurityExtendedRule",
    	"cdsSecurityExtRuleXRef","cdsSecurityRule","cdsSecurityResource"})
    public CSSBootstrap15 securityBootstrap()
    {
    	String clientPropPath = cxsProperties.getSecurity().getDevFramework().getClientPropertiesPath();
    	String securityPropPath = cxsProperties.getSecurity().getDevFramework().getSecurityPropertiesPath();
    	LOGGER.debug("Bootstrapping DevFramework with clientPropertyFilePath=[" + clientPropPath
    			+"] and securityPropertyFilePath=[" + securityPropPath +"]");
    	CSSBootstrap15 bs15 = new CSSBootstrap15();
    	bs15.setClient(true);
    	bs15.setService(true);
    	bs15.setClientProperties(clientPropPath);
    	bs15.setSecurityProperties(securityPropPath);
    	return bs15;
    }
    @Bean
    public CdsClient cdsClient(WebServiceTemplate wsTemplate)
    {
    	CdsClient cdsClient = new CdsClient();
    	cdsClient.setWebServiceTemplate(wsTemplate);
    	return cdsClient;
    }
    @Bean
    public CdsSecurityBase cdsSecurityBase(CdsClient cdsClient)
    {
    	CdsSecurityBase cdsSecurityBase = new CdsSecurityBase();
    	cdsSecurityBase.setCdsClient(cdsClient);
    	return cdsSecurityBase;
    }
    @Bean
    public SaajSoapMessageFactory messageFactory()
    {
    	return new SaajSoapMessageFactory();
    }
    @Bean
    public WebServiceTemplate wsTemplate(WebServiceMessageFactory messageFactory,
    		Jaxb2Marshaller cdsMarshaller)
    {
    	WebServiceTemplate wsTemplate = new WebServiceTemplate();
    	wsTemplate.setMessageFactory(messageFactory);
    	wsTemplate.setDefaultUri(cxsProperties.getSecurity().getDevFramework().getCds().getDefaultUri());
    	wsTemplate.setMarshaller(cdsMarshaller);
    	wsTemplate.setUnmarshaller(cdsMarshaller);
    	wsTemplate.setInterceptors(new ClientInterceptor[]{wsSecurityInterceptor(wsTemplate)});
    	return wsTemplate;
    }
    @Bean
    public ClientInterceptor wsSecurityInterceptor(WebServiceTemplate wsTemplate)
    {
    	SpringClientWsSecurityTokenInterceptor interceptor =
    			new SpringClientWsSecurityTokenInterceptor();
    	interceptor.setWebServiceTemplate(wsTemplate);
    	interceptor.setDefaultServiceAppId(
    			cxsProperties.getSecurity().getDevFramework().getCds().getDefaultServiceAppId());
    	return interceptor;
    }
    @Bean
    public Jaxb2Marshaller cdsMarshaller()
    {
    	Jaxb2Marshaller marshaller = new Jaxb2Marshaller();
    	marshaller.setContextPath("com.fedex.framework.cds");
    	return marshaller;
    }
    @Bean
    public CdsSecurityAction cdsSecurityAction()
    {
    	return new CdsSecurityAction();
    }
    @Bean
    public CdsSecurityRole cdsSecurityRole()
    {
    	return new CdsSecurityRole();
    }
    @Bean
    public CdsSecurityAppRole cdsSecurityAppRole()
    {
    	return new CdsSecurityAppRole();
    }
    @Bean
    public CdsSecurityGroupRole cdsSecurityGroupRole()
    {
    	return new CdsSecurityGroupRole();
    }
    @Bean
    public CdsSecurityUserRole cdsSecurityUserRole()
    {
    	return new CdsSecurityUserRole();
    }
    @Bean
    public CdsSecurityExtendedRule cdsSecurityExtendedRule()
    {
    	return new CdsSecurityExtendedRule();
    }
    @Bean
    public CdsSecurityExtRuleXRef cdsSecurityExtRuleXRef()
    {
    	return new CdsSecurityExtRuleXRef();
    }
    @Bean
    public CdsSecurityRule cdsSecurityRule()
    {
    	return new CdsSecurityRule();
    }
    @Bean
    public CdsSecurityResource cdsSecurityResource()
    {
    	return new CdsSecurityResource();
    }
```

#### API Development
##### Hystrix Setup
###### Configuration
 * Add class level annotation @EnableCircuitBreaker
 * Add method level annotation

```
@HystrixCommand(groupKey = "RATCServiceGroupKey",
			commandKey = "RATCServiceGetRatesCommandKey", threadPoolKey = "RATCServiceThreadPoolKey",fallbackMethod="defaultRates")
```
 * Declare command key and thread pool configurations in application yml files

Example:

```
RATCServiceGetRatesCommandKey:
            execution:
                isolation:
                    strategy : THREAD
                    maxConcurrentRequests : 10          
                    thread:
                        timeoutInMilliseconds : 10000
                        interruptOnTimeout : true
                timeout:
                    enabled : true
            fallback:
                enabled : true
            circuitBreaker:
                enabled : true
                requestVolumeThreshold : 20
                sleepWindowInMilliseconds : 5000
                errorThresholdPercentage : 50
                forceOpen : false
                forceClosed : false
            metrics:
                rollingStats:
                    timeInMilliseconds : 10000
                    numBuckets : 10
                rollingPercentile:
                    enabled : true
                    timeInMilliseconds : 60000
                    numBuckets : 6
                    bucketSize : 100
                healthSnapshot:
                    intervalInMilliseconds : 500
            requestCache:
                enabled : true
            requestLog:
                enabled : true
        RATCServiceThreadPoolKey:
            coreSize : 10
            maxQueueSize : 101
            allowMaximumSizeToDivergeFromCoreSize: true
            queueSizeRejectionThreshold : 15
            keepAliveTimeMinutes : 2
            metrics:
                rollingStats:
                    numBuckets : 12
                    timeInMilliseconds : 1440

```

###### Fallback Methods
Fallback methods are triggered based on Hystrix configuration set in above section. Fallback methods can have same request/response arguments as the original method. In addition it can have Throwable parameter to access the underlying exception that caused hystrix failure.

Example:

```
/* Fall Back method to return default rate Response for anonymous shipment.
 * @param shipShipmentInput
 * @param clientContext
 * @param rootCause
 * @return
 */
public RateOutputVO defaultRates(
		ShipShipmentInputVO shipShipmentInput, ClientContextVO clientContext,Throwable rootCause) {
	LOGGER.error("Error fetching Rates from RATC for Anonymous Shipment.", rootCause);
	LOGGER.warn("Fall back method triggered. Returning default Rates");
	RateOutputVO output = new RateOutputVO();
	List<RateReplyDetail> rateReplyDetails = new ArrayList<RateReplyDetail>();
	RateReplyDetail rateReplyDetail = new RateReplyDetail();
	rateReplyDetail.setAnonymouslyAllowable(true);
	rateReplyDetails.add(rateReplyDetail);
	output.setRateReplyDetails(rateReplyDetails);
	return output;
	}
```

##### Corcuit Breaker
Circuit or a connection between caller service and called service can be configured by parameters. Refer [Circuit Breaker](https://github.com/Netflix/Hystrix/wiki/Configuration#CommandCircuitBreaker)

```
circuitBreaker:
	enabled: true
	requestVolumeThreshold: 20
	sleepWindowInMilliseconds: 5000
	errorThresholdPercentage: 50
	forceOpen: false
	forceClosed: false
```

When circuit breaker opens the circuit, HDSB display as:
![Circuit Breaker](/images/standing-up-new-cxs/circuit_breaker.png)

##### Backend Calls
###### FAST - Service Factory Properties in YML Files
 * Add Service Factory level specific properties files under root content folder.
![Service Factory](/images/standing-up-new-cxs/service_factory_properties.png)
 * Define Fast call dom properties and service factory file location in application.yml (Level Specific)

```
fast:
  cnty:
    domeMajorVersion: 3
    domIntermediateVersion: 1
    domMinorVersion: 0
    softwareId: INET
    softwareReleaseVersion: 35
    applicationId: commondatacal
    softwareReleaseRevision: 52
    serviceFactoryFileLocation: content/cnty/servicefactory/properties.L2-wtc
```

###### Error Handling - FAST To CXS Error Code Conversation
 * Define FAST error codes and messages in main application.yml under cxs

```
fast:
  defaultConversationCodeProperties:
    VACS-501: ADDRESS.IS.REQUIRED
    VACS-797: ADULTSIGNATURE.NOT.ALLOWED
    VACS-795: ADULTSIGNATURE.SERVICESELECTED.NOTALLOWED
    VACS-422: AIRBILL.NOT.ALLOWED
    VACS-506: AIRBILL.VACSVALIDATION.FAILED
    VACS-463: AIRPORT.COMMITMENT.NOTNULL
    VACS-1073: ANCILLARYENDORSEMENT.TYPE.ERROR
```
 * Code that retrieves the FAST error code equivalent code and invokes CFMS service to retrieve the error message for the given code.

```
/*
 * @param code  test
 * @return
 */
private String getErrorTextFromCFMS(String code) {
	String cxsErrorCode = (String) CommonAppConfig.getInstance().getDefaultConversionCodeProperties().get(code);
	if(!StringUtils.isEmpty(cxsErrorCode)){
		List<CXSError> errors = new ArrayList<CXSError>();
		errors.add(new CXSError(cxsErrorCode));
		try {
			cfmsService.populateMessagesFromCFMS(TransactionIdProvider.get(), clientContextVO, errors);
		} catch (CFMSClientValidationException | CFMSClientSystemException e) {
			LOGGER.error("Exception while retrieving the error messages from CFMS",e);
		}
	}
		return cxsErrorCode;
}
```

###### Database
