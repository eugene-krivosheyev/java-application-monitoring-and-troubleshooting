Java Application Monitoring and Troubleshooting Basics
======================================================
_4. Java Application as a Runtime White Box: App running, JVM and application monitoring, troubleshooting, faults analysing and tuning._ 24 hrs / 3 days.

> You have Java application running at prod server, Prometheus and ssh terminal.
> Let's start monitoring!

# Training Objectives
- [ ] Understanding modern application architecture and defect hotspots
- [ ] Understanding JVM classes, memory and threading architecture
- [ ] Hands-on skill of monitoring modern applications
- [ ] Understanding modern IO architecture and its pitfalls
- [ ] Hands-on skill of monitoring persistent data-driven applications

# Prerequisites
## Hardware
- [ ] RAM ≥ 8Гб
- [ ] Wi-Fi with Internet access

## Software at student's developer desktop
- [ ] [Git](https://git-scm.com)
Installation at Windows:
1. Download distribution: [Git for Windows Setup](https://git-scm.com/download/win)
1. Run distribution
1. All installation questions and options leave as default 

- [ ] [AdoptOpenJDK](https://adoptopenjdk.net) OpenJDK 8 HotSpot Latest Release
Installation at Windows:
1. Download distribution: preffered Windows Installer at corporate repo (`softwarecenter:SoftwareID=ScopeId_1999C8B9-1E4A-4832-84B4-AAC87980BDC9/Application_c0cf2d23-58dd-4798-a91c-696b82180ff3`) or fallback [Windows installer at official website](https://github.com/AdoptOpenJDK/openjdk8-binaries/releases/download/jdk8u262-b10/OpenJDK8U-jdk_x86-32_windows_hotspot_8u262b10.msi) 
1. Run distribution
1. All installation questions and options leave as default 

- [ ] [VisualVM](https://visualvm.github.io) latest
Installation at Windows:
1. Download distribution: [default latest version](https://github.com/visualvm/visualvm.src/releases/download/2.0.3/visualvm_203.zip)
1. Unpack distribution to folder by your choice (e.g. "C:\Program Files\")

- [ ] [IntelliJ IDEA CE](https://www.jetbrains.com/idea/download) latest Community Edition
Installation at Windows:
1. Download distribution: [latest Windows Community edition](https://download.jetbrains.com/idea/ideaIC-2020.1.4.exe)
1. Run distribution
1. All installation questions and options leave as default 

- [ ] SSH terminal (for Windows only)
1. Check if you already have SSH client: 
1. run "cmd" program
1. try run `ssh -V`

If you don't have pre-installed SSH client, installation [PuTTY for Windows](https://www.chiark.greenend.org.uk/~sgtatham/putty) at Windows:
1. Download distribution: [MSI Windows Installer 64-bit](https://the.earth.li/~sgtatham/putty/latest/w64/putty-64bit-0.74-installer.msi)
1. Run distribution
1. All installation questions and options leave as default 

- [ ] [JMeter](https://jmeter.apache.org/download_jmeter.cgi)
Installation at Windows:
1. Download distribution: [latest Binary ZIP](https://apache-mirror.rbc.ru/pub/apache//jmeter/binaries/apache-jmeter-5.3.zip)
1. Unpack distribution to folder by your choice (e.g. "C:\Program Files\")
1. Remember this path, later we'll refer it as JMETER_HOME. Or add Windows system variable JMETER_HOME with value of path to unpacked distribution folder.   
1. Add full path to "bin" subfolder to Windows system PATH variable

## Network access from student stations _to_ emulation of **prod** host
- [ ] [prod host](/iaac/inventories/production/hosts.yml) accessible
- [ ] Ports at {{ prod }}[:ports_needed](/iaac/inventories/test/test-env-docker-compose.yml) accessible

# Agenda (20 a.hr.)
## Training introducing and focusing (15m)
- [ ] Schedule
- [ ] Trainer
- [ ] Training overview
- [ ] Rules

## Hands-on: Teams and their demand (15m)
- [ ] Pairs forming and introduction
- [ ] Attendees prerequisites check
- [ ] Topics focus demand from attendees
- [ ] Additional topics demand form attendees

## Java app development crash course (50m)
### What do any application doing?
![System as Public service](/visuals/restaurant-as-system-metaphor.jpg "System as Public service")

| Concept | Metaphor | Code
|---------|----------|-----
| Thread | **Worker man** | Thread created by runtime: `java MyApplication` 
| Thread Pool | [Workers **Team**](/visuals/restaurant-with-thread-pools.jpg) often the same Role | Typical architecture: pool managed by container, async arch: some pools managed by framework/application   
| Data input | **Visitor's wishes** | [Console user input](/src/main/java/com/acme/dbo/Presentation.java#L31)
| Data processing | **Meal recipes, conversation scripts, labor instructions** | [Code as instructions](/src/main/java/com/acme/dbo/Controller.java) 
| Data storing | **Persistent production store** | [Files as persistent store](/src/main/java/com/acme/dbo/ClientRepository.java)
| Data output | **Giving away to Visitor his meals** | [Console output](/src/main/java/com/acme/dbo/Presentation.java#L33)

### How we do model the data?
| Concept | Metaphor | Code
|---------|----------|-----
| Primitive Types | People can think and communicate only with **numbers and strings** | [String restaurant menu](/src/main/java/com/acme/dbo/Presentation.java#L32)
| Structures | People can think with composite entities, **concepts** | [Domain class and enum](/src/main/java/com/acme/dbo/Client.java)
| Object of structure | **Instance** of concept, with its own state differs from other instance | [Dealing with particular object while processing request](/src/main/java/com/acme/dbo/Service.java) 

### How we do model the behavior?
| Concept | Metaphor | Code
|---------|----------|-----
| Procedure | Meal **recipe** or conversation **script** | [Setting behavior with methods](/src/main/java/com/acme/dbo/Presentation.java#L30)
| Call stack | **Chain of actions** workers call at others  | [Calling method from method](/src/main/java/com/acme/dbo/Presentation.java#L12)
| Class | **Role**: Chief or Waiter, state + bunch of procedures dealing with it | Today we likely divide state and behavior to [domain entities](/src/main/java/com/acme/dbo/Client.java) and [services](/src/main/java/com/acme/dbo/Service.java)
| Object of class | **Johnny the Chief** and **Maggy the chief** differs with its state but have same behavior |  
| Application logic | Scenario how to behave all the workers in any case | [Workers takes responsibilities on them to rule at their level](/src/main/java/com/acme/dbo/Controller.java)

### Where data is stored? Core data scopes
| Concept | Metaphor | Implementation
|---------|----------|---------------
| Local/method/stack variables | Short-term memory: Chief remember sugar doze only when doing sugaring | Call Stack   
| Parameters | Details when asking others to do some work: waiter asks johnnyChief.makeMeal(whatMeals?) | Call Stack 
| Object state | State of worker or structure: its current properties values | Heap object space
| - Request scope | Some object state accessible to all the workers in call chain handling request: sticky note or voice message given each worker to next, "not spicy" | Parameters, framework support, ThreadLocal  
| - Session scope | Some object state accessible to all the workers handling all requests from the same Visitor: "its for table 13" | Framework support
| - Singleton/application scope | Some object state accessible to all the workers | Framework support, Language support for static variables
| Persistent | Long-term data store surviving system restarts | File, embedded/local database, remote filesystem, remote database
| Integration | Data stored and processed by external system | Remote system procedure call, message queue

### How do we implement application with Java
| Concept | Metaphor | Reality
|---------|----------|--------
| Runtime | If Developer is CEO setting application logic, Runtime is your vice | JVM API and system library API
| Working with _thread_: Thread API, states, pooling | We can create work force on demand to execute our instructions | But we have some RAM memory and performance cost 
| Working with _class_: dynamic classloading | Instructions what to do workers get just in time not ahead but worker remember it till die | But we have run-time latency costs
| Working with _instance_: create and GC | We ask our vice to hire and retire workers | Objects state costs us RAM memory. When object's no longer needed it purged from RAM

## Teamwork: What metrics do we consider for dev, test, qa and production environments? (30m)
- [ ] What Quality Attributes/NFRs does JVM provide for application?
- [ ] What Quality Attributes/NFRs do we satisfy with application monitoring?

## Java app delivery crash course (50m)
### How do we build Java application?
- [ ] JVM vs JRE vs JDK
- [ ] Physical point ov view for java application
- [ ] Classes, packages and JARs
- [ ] classpath x2
- [ ] Build cycle raw
- [ ] Build cycle with Maven

### How do we run Java application?
- [ ] JVM vs JRE vs JDK
- [ ] Run with JVM
- [ ] Ways for application run-time parameterization: jvm parameters, program arguments, sys/app properties
- [ ] Key JVM parameters for memory setup

### How do we monitor Java application internals?
- [ ] JMX simple tooling demo: JVisualVM
- [ ] JMX architecture overview

## Hands-on quest: Simple application _local_ building, running and monitoring (50m)
### Given
- [ ] Satisfied [prerequisites](#Prerequisites) 
- [ ] Cloned locally [training content](https://bitbucket.raiffeisen.ru/projects/JVMTRAIN/repos/java-application-monitoring-and-troubleshooting/browse)
```shell script
git clone --depth 1 --branch master https://bitbucket.raiffeisen.ru/scm/jvmtrain/java-application-monitoring-and-troubleshooting.git
cd java-application-monitoring-and-troubleshooting
```

- [ ] Credentials for corporate Maven Artifactory repo set up
```shell script
cp iaac/roles/maven/files/settings.xml $M2_HOME/conf/
cp iaac/roles/maven/files/settings-security.xml ~/.m2/
mvn --encrypt-master-password {{ trainer_given_master_password }}
vi ~/.m2/settings-security.xml
```

- [x] Set up Maven to ignore SSL cert errors
```shell script
-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true -Dmaven.wagon.http.ssl.ignore.validity.dates=true
```

### When
- [ ] Project application built locally with IDEA's built-in maven `mvn clean verify [-DskipTests]`
- [ ] Project application ran locally with CLI
```shell script
java \
  -Xms128m -Xmx256m \
  -cp target/dbo-1.0-SNAPSHOT.jar \
  -Dapp.property=value \
    com.acme.dbo.Presentation \
      program arguments
```

- [ ] JVisualVM profiler ran `$JAVA_HOME/bin/jvisualvm`
- [ ] JVisualVM profiler connected to running app `Local connections`
- [ ] OS-specific monitoring tool shows application process details
```shell script
linux$ top [-pid jvmpid]
windows> taskmgr
```

### Then answered and reviewed at debrief
- [ ] What is the default encoding for I/O?
- [ ] Does the maximun heap size shown with JVisualVM correspond to -Xmx key?
- [ ] What is the default heap size for app running?
- [ ] How many java threads is active within JVM? 
- [ ] How many OS threads is active within OS JVM process? 
- [ ] What is the minimal possible heap size for app running?
- [x] What is the difference for profiler times: Self time/Total time, CPU time?

### After debrief
- [ ] Application architecture overview

---

## Modern applications architecture and deployment: What tiers do we monitor? (1h)
| Tier 
|------
| Application logic layers: UI/P, API/C, BL/S, DAL/R
| Application caching 
| Thread Pool 
| JPA Caching
| JPA subsystem 
| Connection Pools
| JDBC subsystem 
| Framework configuration with profiles
| Framework for Spring modules management  
| Framework for Web/SOAP/REST application expose
| Framework for Application
| Application Server/Servlet Container 
| JVM: application debug API
| JVM: application profiling API
| JVM: universal monitoring API
| JVM: threads, IO
| JVM: memory, GC
| JVM: process 
| Container: Networking
| Container: Core
| Message queues
| DBMS
| OS: Threads
| OS: Processes
| Hardware: HDD/SSD
| Hardware: RAM
| Hardware: CPU

[Tiers and components to monitor diagram](http://www.plantuml.com/plantuml/svg/jPRTRzis4C3V_LUmxju6M_ju6uQYGJfi0ResIDQdrGXEv5bcGf4SISdr1l_l8oMvD2T5OrF4JvBkutxtSQHVMmV6DRLOl62uvHB9rUNxTnzktfATSzh-L1G6zkkAkrrJDXODLTAXT6kgwi8r6fZviT02P72nubwwsdgDbTM6ExH5-yFw-_Lt1SCsoEpY1PTKD0mR8qw12geOhFKvmktrpTKVlzpcKHWeKQ1aKD0k_eacCWI2uIK1Y64AxxD5Cr1CgmWkcq-p6apHX_znSWyjp2euLLDmiofKLRryiV3vQoLHEhBy-Pi-XrSVvGuCsuF19G5BJY_afU3LpbK63kIV1V6_Fo-lFspwfvkBtuUdtzwy6PxUNhpRBBh794fN1X_a_MtdNbdoSh_Z3WUddMbe3t29XjX6QsNSXaIXDegYjQG62HNMtkqc2jtE8B1fsPOBj0Vhi2QyK6S2YMwlp4FuEX2FgFlMUEpFQ1rz11rLP-IzvSOr8FZVuBYIcrGf9EcfjUQo-ez8jMY4TxkrgO21Wn8iaYNZriFNFkDbef6y9Ec4CVB9tBSr6ShPfHFUHzs8NH5WBH4AM80SoskibJcSrPvyItwzZ8evJNgaMGGqJvIIqiver4fofum79XVNLnCAsYWVwd4vQyfca4NXL4dxnVD53Miaq95qKlPOtcLCIwmHrS63H1gcXNoKi-NgJC9H5zhf55KYMEn0if3Q8_eUYO3MWbCunH1yY_H01JvY4kOcfcrpJ8PcQwM4JTCOAZfJrNjMqlCg12VftjrH7IiumHiSCCDRDDDuntc6hhDKiR0iMmCrTkDMAzO8jEUJtezgnsPr6BUxKYatBDCSuTXhHgixoxHreIHRNZMc_n0AX9_SPC5p4puFy1dpF4Mgfgw57E2KTdvMyttqkupsT6la-uhvvzK_NbHrxQqjLHg_pBAnQ5RztnL6HZkIwqMegl95osppUByq48t6Skl2kpQgvGpDSdPkKm9MTXSHeK1tWIIVFdlPGUlvDSzV-V7wpqkZE7DdzbWMLe4k35etlJLrI_FhEpxKupKrPsRmhoOG6DMui5LvWJ87UUQM3QOecLOypc5-2Qlqr3brF_kCINQxcyl0CupcUKhd7hCSPYbfe0y-0bbsaO5IjFqL2OVXNm03aTlxf9bQsL3WmoRBwFXbj6LZkGoeGbMSfi3bKtWjcfPJV1POq3mbf4yX3DUZKKAWUHP8RT2YTDqGfxJwARGjntqkPvRQq22oUaNYRENDUAnSi1uSPLc6B89DzF3LntE-PFhhSRpRKUaO1E73SV4Q9MjgyIy0)
<details>
<summary>puml</summary>

```puml
@startuml
!define ICONURL https://raw.githubusercontent.com/tupadr3/plantuml-icon-font-sprites/v2.1.0/devicons
!includeurl ICONURL/coda.puml
!define SPRITESURL https://raw.githubusercontent.com/rabelenda/cicon-plantuml-sprites/v1.0/sprites
!includeurl SPRITESURL/server.puml
!includeurl SPRITESURL/linux.puml
!includeurl SPRITESURL/docker.puml
!includeurl SPRITESURL/java.puml
!includeurl SPRITESURL/tomcat.puml
!includeurl SPRITESURL/cog.puml


component "<$server>\nhardware" as hardware #lightgray {
    [CPU]
    [RAM]
    [HDD]
    [LAN]

    component "<$linux>\nOS" as os #white {
        [container support] 
        [process management]
        [thread management]
        [filesystem i/o]
        [network i/o]

        component "<$docker>\ncontainer" as container #lightgray {
            [network virtualization]
            [port mapping]
            [overlay fs]
            database "disk image"
            
            component "<$java>\njvm process" as jvm #white {
                [class loading]
                [memory management + GC]
                [thread management]
                [filesystem i/o api]
                [network i/o api]
                [monitoring API]
                [profiling API]
                [dubug API]

                component "<$tomcat>\nservlet container" as web_container #lightgray {
                    [tcp connection \n management]
                    [http protocol \n handling]
                    [web application \n lifecycle]
                    [java components \n lifecycle]
                    [thread pools \n management]

                    component "jdbc connection pool" as container_cp {
                        [jdbc driver]
                    }

                    component "<$coda>\nframework modules management system" as spring_boot #white {
                        [framework modules \n management]
                        [application \n configuration context \n management]

                        component "<$coda>\napplication framework" as spring_core #lightgray {
                            [application configuration \n handling]
                            [application configuration \n profiles support]
                            [application components \n management]
                            [common scopes \n management]
                            [user-defined thread pools \n management]
                            [logging \n management]

                            component "jpa persistent provider" #white {
                                [db data caching \n management]
                                component "jdbc connection pool" as app_cp {
                                    [jdbc driver]
                                }
                            }

                            component "<$coda>\nweb/soap/rest framework" as spring_mvc #white {
                                [http protocol \n API]
                                [request routing]
                                [http scopes \n management]
                                [monitoring \n endpoint]

                                component "<$cog>\napplication" as app #lightgray {
                                    [app data \n caching management] #lightgray 
                                    
                                    package "data access \n layer" as dal #white {
                                        [repository]
                                    }
                                    package "business logic \n layer" as bl #white {
                                        [service]
                                    }
                                    package "api \n layer" as cl #white {
                                        [controller]
                                    }
                                    package "presentation \n layer" as pl #white {
                                        [view]
                                    }

                                    service -> repository 
                                    controller -> service
                                    view -> controller
                                }
                            }
                        }
                    }
                }     
            }
        } 
    }
}
@enduml
```
</details>

## Monitoring architecture overview (30m)
![Inrastructure overview](http://www.plantuml.com/plantuml/svg/NL71Ri8m3BtxAopk_0BYmddIs0rLA6sjaQsDepXbe8b_dvCcqEvGANxFVdv-tKiq8KxZ08vxmarFLrGeMdORD89Qbn9_0KyY5umKHklCq2I-5ieP9xOT2FrWxYtpT-OBg7GR-4SeDFhy6Ycc5rGQjxkNqWF0T943ldb94km5zCEjDbMvMZ-7Ab2sFEjv5SpN-S1zFYDNTFCSbCm-4tAk201sF7fsvABUvVrWkvn1awRnDBApaODrdqYkzsnTp2851toi5NIlshZG9BD0Rw9fZH0g6cmdvuiLdRlFWw_kT44LNUEx_uRBeObP-8Lrcx8v0q1OTkKrfyDDEpHAV3ySFr0plDgEy1Ydjac_f2QDsofIeYv0zRQ0rrRs6jKq3dy0)
<details>
<summary>pUML source</summary>

```puml
@startuml
node "dev station" as devstation {
 [ssh terminal] as terminal
 [ansible playbook] as ansible
 [browser]
 [jmeter]

 ansible -> terminal
}

actor Ops as ops
ops --> ansible
ops --> terminal
ops --> browser
ops --> jmeter

node prod {
 [node exporter] as node_exporter

 component [application] {
  [monitoring endpoint] as monitor
 }

 component [prometheus] {
  database metrics_history
 }

 prometheus --> monitor
 prometheus -> node_exporter

 jmeter -> application
 node_exporter -> prod
 
 application --> [External REST service Stub]
}

terminal --> prod
browser --> prometheus
browser --> application
@enduml
```

</details>

## Demo: monitoring environment (15m)
- [ ] Node Exporter
- [ ] Prometheus
- [ ] Grafana: setting up datasource and export dashboard
- [ ] Custom Grafana dashboard as training outcome
- [ ] Adding metrics to Custom Grafana dashboard

## Load generation architecture overview (15m)
- [ ] Types of performance testing except stress testing?
- [ ] While monitoring: What type should we use? What performance metrics do we test?
- [ ] Testing vs Monitoring

## Demo: load generation tools (15m)
- [ ] JMeter overview
- [ ] Test plan
- [ ] Constants
- [ ] Test plan elements
- [ ] Running test plan: GUI and CLI modes
- [ ] Statistics overview: GUI and CLI mode

## Hands-on quest: Modern application _remote_ building, running and monitoring (50m)
### Given
- [ ] SSH user session with domain account to [{{ prod host }}](iaac/inventories/production/hosts.yml) `ssh account@s-msk-t-jvm-XXX`

- [ ] [Demo Application](https://bitbucket.raiffeisen.ru/projects/JVMTRAIN/repos/agile-practices-application/browse) codebase cloned remotely
```shell script
cd /opt
sudo git clone --depth 1 --branch master https://bitbucket.raiffeisen.ru/scm/jvmtrain/agile-practices-application.git
sudo chown {{ account }}:users -R agile-practices-application
cd agile-practices-application
```

- [ ] Made Maven able to run with given user
```shell script
chmod a+x -R /opt/maven
```

- [ ] Credentials for corporate Maven Artifactory repo set up
```shell script
mkdir ~/.m2
cp /opt/maven/settings-security.xml ~/.m2/
mvn --encrypt-master-password {{ trainer_given_master_password }}
vi ~/.m2/settings-security.xml
```

- [ ] Demo Application built remotely
```shell script
cd /opt/agile-practices-application
mvn clean verify [-DskipTests]
```

- [ ] External Legacy System REST stub started
```shell script
cd target/test-classes # cat mappings/legacyAccountingSystemResponse.json
java -jar wiremock-jre8-standalone-2.27.1.jar --port 8888 [--verbose] & # curl localhost:8888/api/account
``` 

- [ ] Application ran at {{ prod }}
```shell script
cd /opt/agile-practices-application
rm -rf dbo-db
nohup \
  java \
    -Xms128m -Xmx128m \
    -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=heapdump.hprof \
    -XX:+TraceClassLoading -XX:+TraceClassUnloading \
    -Xloggc:gc.log -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=8 -XX:GCLogFileSize=8m \
    -XX:NativeMemoryTracking=detail \
    -Dderby.stream.error.file=log/derby.log \
    -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.ssl=false -Dcom.sun.management.jmxremote.authenticate=false \
    -Djava.rmi.server.hostname={{ prod }} -Dcom.sun.management.jmxremote.port=9999 -Dcom.sun.management.jmxremote.rmi.port=9999 \
    -jar target/dbo-1.0-SNAPSHOT.jar \
      --spring.profiles.active=qa \
      --server.port=8080 \
> /dev/null 2>&1 &
```

- [ ] Local load emulation set up: database test data provisioning (`dbo-db` folder)
```shell script
cd java-application-monitoring-and-troubleshooting
jmeter -t load.jmx -j log/jmeter/jmeter.log # GUI mode
```
- Read _constants_ section
- Set up ${PROD_HOST} constant
- Toggled on _setup_ test plan entry
- Shown entry _setup_/Summary Report
- Menu: Run/Start
- Wait while got samples of ${CLIENTS} constant count
- Toggled off _setup_ test plan entry
- Toggled on _reporting-users_, _admin-users_, _operations-users_ test plan entries
- Test plan saved

- [ ] Local load emulation ran
```shell script
cd java-application-monitoring-and-troubleshooting
mkdir -p log/jmeter/report
jmeter -n -t load.jmx -j log/jmeter/jmeter.log -l log/jmeter/jmeter.jtl -e -o log/jmeter/report # CLI mode
```

### When
- [ ] CLI tools used at {{ prod }}
```shell script
dnf install net-tools sysstat

uname --all
cat /etc/os-release
uptime

df -ah
free -m

ps -ef
ps -eaux --forest
ps -eT | grep <pid>

top + 'q'
top + 'f'
top -p <pid>
top -H -p <pid>

vmstat 1 [-w] # mpstat 1
iostat 1 [-xm]
pidstat 1
netstat 1 [-tulnp]

jps [-lvm]
jcmd <pid> help
jcmd <pid> VM.uptime
jcmd <pid> VM.system_properties
jcmd <pid> VM.flags

java -XX:+UnlockDiagnosticVMOptions -XX:+UnlockExperimentalVMOptions -XX:+PrintFlagsFinal -version
jinfo <pid>
jinfo -flag PrintGCDetails <pid> # get jvm flag value
jinfo -flag +PrintGCDetails <pid> # change flag value, makes sense only for _manageable_ ones
```

- [ ] Web applications used
```
http://{{ prod }}:8080/dbo/swagger-ui.html

http://{{ prod }}:8080/dbo/actuator/health
http://{{ prod }}:8080/dbo/actuator
http://{{ prod }}:8080/dbo/actuator/metrics
http://{{ prod }}:8080/dbo/actuator/metrics/jvm.memory.max?tag=area:nonheap&tag=id:Metaspace

http://{{ prod }}:8080/dbo/actuator/prometheus

http://{{ prod }}:9090/alerts
http://{{ prod }}:9090/graph
http://{{ prod }}:9090/graph?g0.range_input=15m&g0.tab=0&g0.expr=http_server_requests_seconds_count

http://{{ prod }}:3000
```

### Finally
- [ ] JMeter load emulation stopped at dev station after ${TEST_DURATION_SEC}
- [ ] Application gracefully stopped at {{ prod }} `curl --request POST http://{{ prod }}:8080/dbo/actuator/shutdown`

### Then answered and reviewed at debrief
- [ ] Free HDD space? Free RAM?
- [ ] How many JVMs running?
- [ ] What DBMS used for application?
- [ ] What JVM version used for application? What are the parameters, properties and arguments used?
- [ ] What are the `health` indicator for application?
- [ ] What is the application uptime?
- [ ] What is the CPU usage for application?
- [ ] How many http requests servlet container handled by different URLs? 
- [ ] How many http sessions are active?
- [ ] What is the current `system load average`?
- [ ] What is the 90% percentile of service response time?

### After debrief
- [ ] Updated your custom Grafana dashboard with metrics you think is important `http://{{ prod }}:3000`
- [ ] Recommendations on informational architecture  
  
---

## Modern applications architecture and deployment: How do we monitor tiers? (1h)
| Tier | Implementation | Tools
|------|----------------|------
| Application Layers | PWA or Server-side Template Engine, Spring @Controllers, @Services, Spring Data JPA @Repositories | [Spring Metrics for Counters, Timers, Long Task Timers, Statistics](https://docs.spring.io/spring-metrics/docs/current/public)
| Application caching | spring-boot-starter-cache module + built-in default Simple cache provider | [Spring Metrics for Caches](https://docs.spring.io/spring-metrics/docs/current/public/prometheus#caches)
| Thread Pool | Java built-in ExecutorService | [Spring Metrics for DataSources](https://docs.spring.io/spring-metrics/docs/current/public/prometheus#executor-services)
| JPA subsystem and JPA Caching | Hibernate | [service:jmx://](https://vladmihalcea.com/hibernate-statistics-jmx/) [Hibernate built-in statistics](https://vladmihalcea.com/hibernate-statistics/)
| JDBC subsystem and Connection Pools | Derby JDBC driver + HikariCP | [service:jmx://com.zaxxer.hikari](https://github.com/brettwooldridge/HikariCP/wiki/MBean-(JMX)-Monitoring-and-Management), [Spring Metrics for DataSources](https://docs.spring.io/spring-metrics/docs/current/public/prometheus#data-sources)
| Framework for modules management | Spring Boot | [spring-boot-actuator](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html) + Built-in Micrometer + [Prometheus Adapter](https://mvnrepository.com/artifact/io.micrometer/micrometer-registry-prometheus) 
| Framework for Application | Spring Core + Spring MVC (spring-boot-starter-web) | [Spring Metrics for Web Instrumentation](https://docs.spring.io/spring-metrics/docs/current/public/prometheus#web) [for Prometheus], Core [Micrometer](http://micrometer.io) [for Prometheus]
| Application Server/Servlet Container | spring-boot-starter-tomcat | 
| JVM: application debug API | JPDA | [jsadebugd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jsadebugd.html)
| JVM: application profiling API | JVMTI | [hprof](https://docs.oracle.com/javase/8/docs/technotes/samples/hprof.html)
| JVM: threads, IO | JVM scheduler, JNI | [jstack](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstack.html)
| JVM: memory, GC | Built-in Garbage Collectors | [jstat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html), [jstatd](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstatd.html), [jmap](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jmap.html), [jhat](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jinfo.html) removed at jdk9
| JVM: universal monitoring API | [JMX](https://docs.oracle.com/javase/tutorial/jmx/index.html) | [jvisualvm](https://docs.oracle.com/javase/8/docs/technotes/guides/visualvm/index.html) 
| JVM: process | Oracle/OpenJDK JRE | [jps](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jps.html), [jcmd](https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/tooldescr006.html), [jinfo](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jinfo.html)
| Containers | Docker | [docker cli](https://docs.docker.com/config/containers/runmetrics/), [docker api for Prometheus](https://docs.docker.com/config/daemon/prometheus/), [Prometheus cAdvisor](https://prometheus.io/docs/guides/cadvisor/)
| Message queues | n/u | vendor tools, prometheus exporters
| DBMS | Apache Derby / Postgresql | vendor tools, [Prometheus pg_exporter](https://github.com/wrouesnel/postgres_exporter), [pg explain](https://postgrespro.ru/docs/postgresql/9.6/sql-explain), [pg analyse](https://postgrespro.ru/docs/postgresql/9.6/sql-analyze)
| OS | Linux | [ps](https://www.geeksforgeeks.org/ps-command-in-linux-with-examples/), [top](https://www.geeksforgeeks.org/top-command-in-linux-with-examples/)
| Hardware | x86 | `df`, `free`, [SNMP](https://docs.oracle.com/javase/8/docs/technotes/guides/management/snmp.html), [Prometheus Node Exporter](https://prometheus.io/docs/guides/node-exporter/)

## Teamwork: What metrics do we consider for dev, test, qa and production environments? (15m)
- [ ] Adding metrics to Custom Grafana dashboard

## JIT compilation (45m)
### JIT
- [ ] ~~javac~~
- [ ] JIT compilation and _compilation eligibility_
- [ ] Code Cache Memory: JIT native code
- [ ] C1 and C2 JIT compilers
- [ ] Compilation threads: C1(1/3) + C2(2/3)
- [ ] On-stack replacement (OSR) and _background compilation_ queue
- [ ] C1 & C2 compiler queues dynamically resizes depending on available %CPU
- [ ] Tiers:
```
0: interpreted mode
1: simple C1 compiled code 
2: limited C1 compiled code 
3: full C1 compiled code 
4: C2 compiled code
```
- [ ] Performance-happy path: 0 -> 3 -> 4
- [ ] Deoptimization -> level 0 (_not entrant_ or _zombie_ code)
### Optimization examples
- [ ] Dead code elimination
- [ ] Inlining
- [ ] C2 Escape analysis for var caching, synchronization, skipping object creation
- [ ] Intel SSE and AVX instruction set 
### Tiered compilation trade-offs
- [ ] Classes compiled, commited code cache
- [ ] Startup time
- [ ] Application throughput as f(warmup time)

## Hands-on quest: JIT compilation monitoring (50m)
### Given
- [ ] Application ran at {{ prod }}
- [ ] External Legacy System REST stub started
- [ ] Load emulation ran

### When
- [ ] CLI tools used at {{ prod }}
```shell script
java -XX:+UnlockDiagnosticVMOptions -XX:+UnlockExperimentalVMOptions -XX:+PrintFlagsFinal -version | grep CodeCache
jinfo -flag UseCodeCacheFlushing <pid>
jinfo -flag ReservedCodeCacheSize <pid>

jinfo -flag CICompilerCount <pid>
jinfo -flag BackgroundCompilation <pid>

jinfo -flag TieredCompilation <pid>
jinfo -flag CompileThreshold <pid> # applies only when standard compilation: -XX:-TieredCompilation
jinfo -flag Tier3InvocationThreshold <pid> # applies when tiered compilation: -XX:+TieredCompilation
jinfo -flag Tier4InvocationThreshold <pid> # applies when tiered compilation: -XX:+TieredCompilation

jstat -compiler <pid>
jstat -printcompilation <pid> [1000]
```

- [ ] Web applications used
```
http://{{ prod }}:8080/dbo/actuator/metrics
http://{{ prod }}:9090/graph
```

- [ ] Profiler used
```shell script
jconsole://{{ prod }}:9999/Memory/Code cache
jconsole://{{ prod }}:9999/MBeans
```

### Finally
- [ ] JMeter load emulation stopped
- [ ] Application gracefully stopped
- [ ] Database filled up with tests data removed

### Then answered and reviewed at debrief
- [ ] Compiled classes number 
- [ ] Total compilation time
- [ ] Is Code Cache full enough to begin worry about `CodeCache is full. Compiler has been disabled.`?
- [ ] Is there Code Cache rolling after warm-up?

### After debrief
- [ ] Updated your custom Grafana dashboard with metrics you think is important

---

## JVM memory management (2h)
### JVM memory architecture
![Hotspot JVM Architecture Overview](https://ionutbalosin.com/wp-content/uploads/2020/01/HotSpotArchitecture.png)
- [ ] [Non-heap] Code Cache: JIT native code, thread stacks, native libraries 
- [ ] [Non-heap] Allocated native memory: direct allocation, NIO buffers 
- [ ] [Non-heap] Compressed Class space
- [ ] [Non-heap] Perm/Meta space
- [ ] [Heap/Object space] New/Young generation (for some GCs also Eden and Survivor spaces)
- [ ] [Heap/Object space] Old/Tenured generation
![JVM memory overview](visuals/jvm-memory-overview.png)
- [ ] Allocated vs Reserved vs Available memory 

### Teamwork: Where and when is memory allocated in the application code? (15m)
- [ ] For given application codebase spot all the timepoints and places of memory allocation.

### Garbage
- [ ] What is garbage: objects ready for gc?
- [ ] Stack trace
### Generational GC algorithms
- [ ] Minor VS Full GC and theirs ratio
- [ ] Copying collector for young generation
- [ ] Mark-sweep-compact (MSC) collector for old generation
### Concurrent GC algorithms
- [ ] _Stop-the-world_ pauses
- [ ] _Concurrent_/low-pause collectors: without stopping _application threads_ when it possible (e.g. Mark phase)
- [ ] Trade-off: low-pauses vs CPU utilization taken out from application
### [HotSpot collectors](https://docs.oracle.com/en/java/javase/12/gctuning/)
| Collector name | Main idea | Full support /deprecation /experimental in JDK8 | JDK 11 | JDK 12 | Settings
|----------------|-----------|-------------------------------------------------|--------|--------|----------
| Serial | *Single-threaded* for new and old gen. *S-t-w* for full and minor gc. *MSC* for new and old gen. Suitable for containerized apps.| + | + | + | `-XX:+UseSerialGC`
| Concurrent Mark-Sweep (CMS) | *Multi-threaded* for new and old gen. *S-t-w* for full and minor gc. *Copying* for new gen, *MSC* for old gen. | + | - | - | `-XX:+UseConcMarkSweepGC` 
| Throughput / Parallel | *Multi-threaded* for new and old gen. *S-t-w* for full and minor gc. *Copying* for new gen, *MSC* for old gen. |  + | + | + | `-XX:+UseParallelGC` `-XX:+UseParallelOldGC`
| G1 | Multi-regional. *Multi-threaded* for new and old gen. *S-t-w* for new gen, *concurrent* for old gen. *Copying* for new and old gen. | +/- | + | + | `-XX:+UseG1GC`
| ZGC | | n/a | ? | ? |
| Shenandoah | | ?? | ?? | ?? |
| Epsilon | | n/a | ? | ? |

### Heap dumps
- [ ] Creating heap dump
- [ ] Analysing heap dump
- [ ] GC roots
- [ ] _Shallow_, _deep_ (aggregation) and _retained_ (composition) sizes
- [ ] _Dominators_ of the heap
- [ ] Objects size and alignment (%8bytes), [jol](https://openjdk.java.net/projects/code-tools/jol) tool
- [ ] Object [reference types](https://dzone.com/articles/weak-soft-and-phantom-references-in-java-and-why-they-matter): hard, soft, weak, phantom
- [ ] _Compressed Oops_ as 35-bit reference (up to 32Gb) stored as 32-bit reference

### Teamwork: What metrics do we consider for dev, test, qa and production environments? (15m)
- [ ] Adding metrics to Custom Grafana dashboard

### _Native/off-heap_ memory more deep dive and new settlers
- [ ] Metaspace
- [ ] Thread stacks
- [ ] Code cache 
- [ ] GC structures
- [ ] Symbols as interned constants
- [ ] Native Byte Buffers: JNI allocations
- [ ] Native Byte Buffers: NIO direct buffers
      

## Hands-on quest: Memory monitoring (50m)
### Given
- [ ] Application ran at {{ prod }}
- [ ] External Legacy System REST stub started
- [ ] Load emulation ran

### When
- [ ] CLI tools used at {{ prod }}
```shell script
jstat -gccapacity <pid>
jstat -gcutil <pid> [1000]
jstat -gc <pid>
jstat -gc <pid> 1000 10

jcmd <pid> GC.heap_info
jinfo -flag DisableExplicitGC <pid>
jcmd <pid> GC.run
jcmd <pid> GC.heap_info

jinfo -flag NewRatio <pid>
jinfo -flag NewSize <pid>
jinfo -flag MaxNewSize <pid>
jinfo -flag MetaspaceSize <pid>
jinfo -flag MaxMetaspaceSize <pid>
jinfo -flag ParallelGCThreads <pid>
jinfo -flag ConcGCThreads <pid> 
jinfo -flag MaxGCPauseMillis <pid> # for Throughput collector 
jinfo -flag InitiatingHeapOccupancyPercent <pid> # for G1 collector 

jcmd <pid> GC.class_histogram # jmap -histo[:live] <pid>

jcmd <pid> GC.heap_dump /tmp/dump.hprof # jmap -dump[:live,format=b],file=/tmp/dump.hprof <pid>
devstation$ scp -P 22 {{ account }}@{{ prod }}:/tmp/dump.hprof .

jcmd <pid> VM.native_memory summary
```

- [ ] Web applications used
```
http://{{ prod }}:8080/dbo/actuator/metrics
http://{{ prod }}:9090/graph
```

- [ ] Profiler used
```shell script
jconsole://localhost:9999/Memory/Chart:*
jconsole://localhost:9999/MBeans

jvisualvm://File/Load (dump.hprof)

jvisualvm://VisualGC (plugin required)
jvisualvm://Buffer Pools (plugin required)
```

### Finally
- [ ] JMeter load emulation stopped
- [ ] Application gracefully stopped
- [ ] Database filled up with tests data removed

### Then answered and reviewed at debrief
- [ ] What GC type is used?
- [ ] Has application stop-the-world GC pauses?
- [ ] Is enough memory for given load?
- [ ] Memory region with most risk of OOME
- [ ] Top memory consuming application classes
- [ ] What are the memory dominators?
- [ ] What is the native memory footprint?

### After debrief
- [ ] Updated your custom Grafana dashboard with metrics you think is important

---

## JVM threading management (2h)
![JVM threads overview](visuals/jvm-threads-overview.png)
### JVM threading architecture
- [ ] Thread definition and duality
- [ ] Scheduler and preemptive concurrency
- [ ] Scheduling overhead
- [ ] Green and native threads
- [ ] Thread resources consumption: heap, stack, CPU, OS limits for threads and native stack memory 
### Thread states
- [ ] Main states and transitions
- [ ] Types of blocking/waiting
### Stack
- [ ] What does thread do? 
- [ ] Stack trace
- [ ] Thread dump
### Thread pools
- [ ] Why thread pools?
- [ ] Pool metrics

### Teamwork: Where and when threads start in the application code? (15m)
- [ ] For given application codebase spot all the timepoints and places of starting thread.

### Typical concurrency issues and solutions
| Issue | WTF | Solutions
|-------|-----|----------
| Data Race | Concurrent non-atomic operation execution | Blocking: synchronized/monitor, non-blocking: atomics/CAS  
| Visibility and Reordering | JVM aggressive optimizations | JMM: synchronized + volatile
| Deadlock | Threads blocked by each other | oops ( 

### Teamwork: Is there correlation between sync code presence and application throughput (15m)
- [ ] Performance = f(% sync code) ?

### Concurrent memory-intensive applications design patterns
- [ ] async operations + thread synchronization: wait/notify
- [ ] threading patterns for logic processing: sync/async with multiple pools
- [ ] MMO game/DBMS cases: entry point synchronization, coarse-grained data objects locking, fine-grained data objects subsequent locking 
- [ ] concurrent data structures

### Teamwork: What metrics do we consider for dev, test, qa and production environments? (15m)
- [ ] Adding metrics to Custom Grafana dashboard

## Hands-on quest: Threads monitoring (50m)
### Given
- [ ] Application ran at {{ prod }}
- [ ] External Legacy System REST stub started
- [ ] Load emulation ran

### When
- [ ] CLI tools used at {{ prod }}
```shell script
top + 'f' -> nTH -> space -> Esc
jstack <pid> > thread-dump.tdump
scp -P 22 {{ account }}@{{ prod }}:/tmp/thread-dump.txt .
```

- [ ] Profiler used
```shell script
jvisualvm://File/Load (thread-dump.tdump)
jvisualvm://Threads (plugin Threads inspector required)
```

- [ ] Thread dump analysed

- [ ] Web applications used
```
http://{{ prod }}:8080/dbo/actuator/metrics
http://{{ prod }}:9090/graph
```

### Finally
- [ ] JMeter load emulation stopped
- [ ] Application gracefully stopped
- [ ] Database filled up with tests data removed

### Then answered and reviewed at debrief
- [ ] How many threads are in system?
- [ ] Native or Green threads implemented by JVM?
- [ ] How many threads working on requests processing?
- [ ] Common thread state at normal load?
- [ ] Is it enough of threads?
- [ ] Where threads count can be adjusted?
- [ ] Is there contended in-memory resource?
- [ ] Hypothesis on what business logic is most CPU consuming
- [ ] Hypothesis on application threading patterns: (a) connection handling, (b) logic processing, (c) data access?

### After debrief
- [ ] Updated your custom Grafana dashboard with metrics you think is important

---

## JVM IO management (1.5h)
### Blocking IO architecture
- [ ] Synchronous IO concept
- [ ] What sources do we use to get/store data?
### Typical issues
- [ ] Excessive IO wrapper classes objects allocation
- [ ] Encoding
- [ ] Buffering as extra-resource allocation, non-reusability with OS
- [ ] Buffering as CAP case
- [ ] Blocking expensive server thread for user data IO 
- [ ] Excessive resource allocation: closing resources
### Outgoing connections pooling
- [ ] Resource pools main metrics

### HTTP protocol
- [ ] HTTP overview
- [ ] Commands and response codes
- [ ] HTTP session concept
- [ ] *Keep-alive*
- [ ] Content zipping 

### Teamwork: What metrics do we consider for dev, test, qa and production environments? (15m)
- [ ] Adding metrics to Custom Grafana dashboard

### Non-blocking IO architecture
- [ ] Asynchronous IO concept
- [ ] NIO overview
- [ ] Threading patterns for connection data processing: sync/async (supported by sync/async Servlets, Spring MVC/WebFlux)
### IO data processing architecture patterns 
- [ ] Threading scope patterns: thread-per-request, thread-per-connection, thread-per-session
- [x] Threading patterns for logic processing: sync/async with multiple pools
- [ ] Data access: sync/async ([files](https://www.baeldung.com/java-nio2-async-file-channel), [DB](https://spring.io/projects/spring-data-r2dbc), [HTTP REST calls](https://dzone.com/articles/high-concurrency-http-clients-on-the-jvm))

## Hands-on quest: IO monitoring (50m)
### Given
- [ ] Application ran at {{ prod }}
- [ ] External Legacy System REST stub started
- [ ] Load emulation ran

### When
- [ ] Profiler used
```shell script
jvisualvm://Threads (plugin Threads inspector required)
```
- [ ] Thread dump analysed
- [ ] Heap dump analysed
- [ ] Web applications used
```
http://{{ prod }}:8080/dbo/actuator/metrics
http://{{ prod }}:9090/graph
```

### Finally
- [ ] JMeter load emulation stopped
- [ ] Application gracefully stopped
- [ ] Database filled up with tests data removed

### Then answered and reviewed at debrief
- [ ] Did test clients get http errors? 
- [ ] Incoming http connections long-liveness
- [ ] What is the system default IO encoding?
- [ ] Is there excessive IO wrapper objects allocation?
- [ ] Is outgoing HTTP connections pooled?
- [ ] Is it enough HTTP connections in pool?
- [ ] What threading pattern for connection data processing used in application design?
- [ ] What threading scope pattern used in application design?

### After debrief
- [ ] Updated your custom Grafana dashboard with metrics you think is important

---

## Persistent data management (1.5h)
### JDBC subsystem architecture
- [ ] JDBC API
- [ ] Driver types
- [ ] Connection lifecycle
- [ ] Prepared statements
- [ ] ResultSet fetching
### DB Connection pooling
- [ ] DB Connection pool architecture
- [ ] Prepare Statements pooling
- [ ] Pool metrics
### JDBC Transactions
- [ ] Transaction concept
- [ ] Isolation levels
- [ ] Transaction resources trade-offs

### Teamwork: What metrics do we consider for dev, test, qa and production environments? (15m)
- [ ] Adding metrics to Custom Grafana dashboard

### JPA architecture
- [ ] JPA API
- [ ] EntityManager and Context/Session
- [ ] [Caching levels](/visuals/jpa-cache-overview.jpg)
- [ ] JPA transactions architecture
### Spring JPA architecture
- [ ] Spring Data JPA module
- [ ] Repositories code generation
- [ ] Spring transaction management

## Hands-on quest: JDBC subsystem monitoring (50m)
### Given
- [ ] Application ran at {{ prod }}
- [ ] External Legacy System REST stub started
- [ ] Load emulation ran

### When
- [ ] Profiler used
```shell script
jvisualvm://Threads (plugin Threads inspector required)
```
- [ ] Thread dump analysed
- [ ] Heap dump analysed
- [ ] Web applications used
```
http://{{ prod }}:8080/dbo/actuator/metrics
http://{{ prod }}:9090/graph
```

### Finally
- [ ] JMeter load emulation stopped
- [ ] Application gracefully stopped
- [ ] Database filled up with tests data removed

### Then answered and reviewed at debrief
- [ ] Is JDBC connections pooled?
- [ ] Is it enough JDBC connections in pool?
- [ ] ORM cache state?
- [ ] Can we set less transaction isolation level to gain throughput?

### After debrief
- [ ] Updated your custom Grafana dashboard with metrics you think is important

---

## Final retro (15m)
- [ ] Value taken
- [ ] Process Improvement Actions 
- [ ] Training Improvement Actions
- [ ] Custom Grafana dashboards competition

---

## Reference
- [jstat GC monitoring](https://dzone.com/articles/how-monitor-java-garbage)
- [HotSpot JVM full options reference](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)
- [10 Important JVM Options](https://geekflare.com/important-jvm-options)
- [Most Important JVM Parameters](https://www.baeldung.com/jvm-parameters)
- [HotSpot JVM Performance Tuning Guidelines](https://ionutbalosin.com/2020/01/hotspot-jvm-performance-tuning-guidelines/)
- [Garbage Collectors Table](http://www.fasterj.com/articles/oraclecollectors1.shtml)
- [Java Performance Book, 2nd edition](https://www.oreilly.com/library/view/java-performance-2nd/9781492056102/)
