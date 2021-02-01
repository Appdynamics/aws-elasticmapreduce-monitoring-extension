# AWS ElasticMapReduce Monitoring Extension

## Use Case
Captures ElasticMapReduce statistics from Amazon CloudWatch and displays them in the AppDynamics Metric Browser.

**Note : By default, the Machine agent can only send a fixed number of metrics to the controller. This extension potentially reports thousands of metrics, so to change this limit, please follow the instructions mentioned [here](https://docs.appdynamics.com/display/PRO40/Metrics+Limits).**

## Prerequisites
1. Please give the following permissions to the account being used to with the extension.
   **cloudwatch:ListMetrics**
   **cloudwatch:GetMetricStatistics**
2. In order to use this extension, you do need a [Standalone JAVA Machine Agent](https://docs.appdynamics.com/display/PRO44/Standalone+Machine+Agents) or [SIM Agent](https://docs.appdynamics.com/display/PRO44/Server+Visibility).  For more details on downloading these products, please  visit [here](https://download.appdynamics.com/).
3. The extension needs to be able to connect to AWS Cloudwatch in order to collect and send metrics. To do this, you will have to either establish a remote connection in between the extension and the product, or have an agent on the same machine running the product in order for the extension to collect and send the metrics.

** Note : This extension is compatible with Machine Agent version 4.5.13 or later.

* If you are seeing warning messages while starting the Machine Agent, update the http-client and http-core JARs in {MACHINE_AGENT_HOME}/monitorsLibs to httpclient-4.5.9 and httpcore-4.4.12 to make this warning go away.
* To make AWS extensions work on Machine Agent < 4.5.13: The http-client and http-core JARs in {MACHINE_AGENT_HOME}/monitorsLibs has to be manually be updated to httpclient-4.5.9 and httpcore-4.4.12


## Installation

1. Run 'mvn clean install' from aws-elasticmapreduce-monitoring-extension
2. Copy and unzip AWSElasticMapReduceMonitor-\<version\>.zip from 'target' directory into \<machine_agent_dir\>/monitors/
3. Edit config.yml file in AWSElasticMapReduceMonitor/conf and provide the required configuration (see Configuration section)
4. Restart the Machine Agent.
Please place the extension in the "**monitors**" directory of your Machine Agent installation directory. Do not place the extension in the "**extensions**" directory of your Machine Agent installation directory.

## Configuration
In order to use the extension, you need to update the config.yml file that is present in the extension folder. The following is a step-by-step explanation of the configurable fields that are present in the `config.yml` file.

1. If SIM is enabled, then use the following metricPrefix - 

   `metricPrefix: "Custom Metrics|Amazon ElasticMapReduce|"`
    
   Else, configure the "**COMPONENT_ID**" under which the metrics need to be reported. This can be done by changing the value of `<COMPONENT_ID>` in
   `metricPrefix: "Server|Component:<COMPONENT_ID>|Custom Metrics|Amazon ElasticMapReduce|"`.

   For example,
         
    ```
    metricPrefix: "Server|Component:100|Custom Metrics|Amazon ElasticMapReduce|"
    ```

2. Provide **accessKey**(required) and **secretKey**(required) of your account(s), also provide **displayAccountName**(any name that represents your account) and
   **regions**(required). If you are running this extension inside an EC2 instance which has **IAM profile** configured then you don't have to configure **accessKey** and  **secretKey** values, extension will use **IAM profile** to authenticate. You can provide multiple accounts and regions as below - 
   ~~~
   accounts:
     - awsAccessKey: "XXXXXXXX1"
       awsSecretKey: "XXXXXXXXXX1"
       displayAccountName: "TestAccount_1"
       regions: ["us-east-1","us-west-1","us-west-2"]

     - awsAccessKey: "XXXXXXXX2"
       awsSecretKey: "XXXXXXXXXX2"
       displayAccountName: "TestAccount_2"
       regions: ["eu-central-1","eu-west-1"]
   ~~~
3. If you want to encrypt the **awsAccessKey** and **awsSecretKey** then follow the "Credentials Encryption" section and provide the encrypted values in **awsAccessKey** and **awsSecretKey**. Configure `enableDecryption` of `credentialsDecryptionConfig` to `true` and provide the encryption key in `encryptionKey`.
   For example,
   ```
   #Encryption key for Encrypted password.
   credentialsDecryptionConfig:
       enableDecryption: "true"
       encryptionKey: "XXXXXXXX"
   ```
4. To report metrics only from specific dimension values, configure the `dimesion` section. Dimensions for AWS EMR are `JobFlowId, JobId`. 
    ```
    dimensions:
      - name: "JobFlowId"
        displayName: "JobFlowId"
        values: [".*"]
    ```
     If `.*` is used, all dimension values are monitored and if empty, none are monitored.
5. Configure the metrics section.
   
    For configuring the metrics, the following properties can be used:

    |     Property      |   Default value |         Possible values         |                                              Description                                                                                                |
    | :---------------- | :-------------- | :------------------------------ | :------------------------------------------------------------------------------------------------------------- |
    | alias             | metric name     | Any string                      | The substitute name to be used in the metric browser instead of metric name.                                   |
    | statType          | "ave"           | "AVERAGE", "SUM", "MIN", "MAX"  | AWS configured values as returned by API                                                                       |
    | aggregationType   | "AVERAGE"       | "AVERAGE", "SUM", "OBSERVATION" | [Aggregation qualifier](https://docs.appdynamics.com/display/PRO44/Build+a+Monitoring+Extension+Using+Java)    |
    | timeRollUpType    | "AVERAGE"       | "AVERAGE", "SUM", "CURRENT"     | [Time roll-up qualifier](https://docs.appdynamics.com/display/PRO44/Build+a+Monitoring+Extension+Using+Java)   |
    | clusterRollUpType | "INDIVIDUAL"    | "INDIVIDUAL", "COLLECTIVE"      | [Cluster roll-up qualifier](https://docs.appdynamics.com/display/PRO44/Build+a+Monitoring+Extension+Using+Java)|
    | multiplier        | 1               | Any number                      | Value with which the metric needs to be multiplied.                                                            |
    | convert           | null            | Any key value map               | Set of key value pairs that indicates the value to which the metrics need to be transformed. eg: UP:0, DOWN:1  |
    | delta             | false           | true, false                     | If enabled, gives the delta values of metrics instead of actual values.                                        |

   For example,
   ```
   - name: "Bytes"
     alias: "Bytes per minute (Unit - byte; StatType - sum)"
     statType: "sum"
     aggregationType: "AVERAGE"
     timeRollUpType: "AVERAGE"
     clusterRollUpType: "INDIVIDUAL"
     delta: false
     multiplier: 1
   ```
   
   **All these metric properties are optional, and the default value shown in the table is applied to the metric(if a property has not been specified) by default.**

### Config.yml
Please avoid using tab (\t) when editing yaml files. Please copy all the contents of the config.yml file and go to [Yaml Validator](http://www.yamllint.com/) . On reaching the website, paste the contents and press the “Go” button on the bottom left.                                                       
If you get a valid output, that means your formatting is correct and you may move on to the next step.

## Metrics
Typical metric path: **Application Infrastructure Performance|\<Tier\>|Custom Metrics|Amazon ElasticMapReduce|\<Account Name\>|\<Region\>|JobFlow Id|\<JobFlow Id\>** followed by the metrics defined in the link below:

- [ElasticMapReduce Metrics](http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/emr-metricscollected.html)

## Credentials Encryption
Please visit [this page](https://community.appdynamics.com/t5/Knowledge-Base/How-to-use-Password-Encryption-with-Extensions/ta-p/29397) to get detailed instructions on password encryption. The steps in this document will guide you through the whole process.

## Extensions Workbench

Workbench is an inbuilt feature provided with each extension in order to assist you to fine tune the extension setup before you actually deploy it on the controller. Please review the following document on [How to use the Extensions WorkBench](https://community.appdynamics.com/t5/Knowledge-Base/How-to-use-the-Extensions-WorkBench/ta-p/30130)

## Troubleshooting

Please follow the steps listed in this [troubleshooting-document](https://community.appdynamics.com/t5/Knowledge-Base/How-to-troubleshoot-missing-custom-metrics-or-extensions-metrics/ta-p/28695) in order to troubleshoot your issue. These are a set of common issues that customers might have faced during the installation of the extension. If these don't solve your issue, please follow the last step on the [troubleshooting-document](https://community.appdynamics.com/t5/Knowledge-Base/How-to-troubleshoot-missing-custom-metrics-or-extensions-metrics/ta-p/28695) to contact the support team.

## Support Tickets

If after going through the [Troubleshooting Document](https://community.appdynamics.com/t5/Knowledge-Base/How-to-troubleshoot-missing-custom-metrics-or-extensions-metrics/ta-p/28695) you have not been able to get your extension working, please file a ticket and add the following information.

Please provide the following in order for us to assist you better.

1. Stop the running machine agent.
2. Delete all existing logs under `<MachineAgent>/logs`.
3. Please enable debug logging by editing the file `<MachineAgent>/conf/logging/log4j.xml`. Change the level value of the following `<logger>` elements to debug.
   ```
   <logger name="com.singularity">
   <logger name="com.appdynamics">
   ```
4. Start the machine agent and please let it run for 10 mins. Then zip and upload all the logs in the directory `<MachineAgent>/logs/*`.
5. Attach the zipped `<MachineAgent>/conf/*` directory here.
6. Attach the zipped `<MachineAgent>/monitors/ExtensionFolderYouAreHavingIssuesWith` directory here.
   
For any support related questions, you can also contact [help@appdynamics.com](mailto:help@appdynamics.com).

## Contributing

Always feel free to fork and contribute any changes directly here on [GitHub](https://github.com/Appdynamics/aws-elasticmapreduce-monitoring-extension).

## Version
   |Name|Version|
   |--------------------------|------------|
   |Extension Version         |1.0.4       |
   |Controller Compatibility  |4.5 or Later|
   |Machine Agent             |4.5.13+     |
   |Last Update               |01/02/2021  |

List of changes to this extension can be found [here](https://github.com/Appdynamics/aws-elasticmapreduce-monitoring-extension/blob/master/CHANGELOG.md)
