# CIBR/FLARE Lake Forecast Automatic Deployment

## IBM Cloud Functions Deployment

This repository holds JSON configuration files for automated deployment of  multiple lake forecasts for the CIBR/FLARE project using OpenWhisk for microservice execution and S3 buckets to store and serve data.

The workflow for forecast users to have a lake forecast automatically deployed is as follows:

### Clone, develop, and test your own code repository
The first requirement is for a user to have a stable, tested code repository with the FLARE codebase applied to your lake. [Here is an example of the repository for FCRE](https://github.com/FLARE-forecast/FCRE-forecast-code). 

Please follow the forecast code repository documentation to develop your own lake, and make sure your code is tested to run locally (in your own container or virtual machine) before proceeding with automated deployment, as tweaking and debugging an automated deployment is significantly more complex than a local deployment.

### Create your json configuration file

To describe an automated deployment, you need to create a .json file with the following format:

```json=
{
  "forecast_code": "XXX",
  "config_set": "default",
  "function": "1",
  "configure_run": "configure_run.yml",
  "use_https": "TRUE",
  "aws_default_region": "s3",
  "aws_s3_endpoint": "flare-forecast.org",
  "schedule": "0 */6 * * *"
}
```

User-provided values (you need to configure these):
* "forecast_code" specifies the URL of your own code repository, e.g. "forecast_code": "https://github.com/FLARE-forecast/FCRE-forecast-code".
* "config_set" specifies the configuration set, which is the name of the directory under "configuration" in your code repository from where to download a configuration file. For example, with "config_set": "default" the configuration files need to be present in "FCRE-forecast-code/configuration/default/" (https://github.com/FLARE-forecast/FCRE-forecast-code/tree/main/configuration/default).
* "configure_run" specifies the YAML configuration file from your run; this file should be present in the repository and config_set as per above. For example, with "configure_run": "configure_run.yml" the file "FCRE-forecast-code/configuration/default/configure_run.yml" (https://github.com/FLARE-forecast/FCRE-forecast-code/blob/main/configuration/default/configure_run.yml) will be used.
* "schedule" specifies when the forecast should run, using the [UNIX cron format](https://en.wikipedia.org/wiki/Cron); for instance, "schedule": "0 */6 * * *" configures your deployment to run every 6 hours, at the top of the hour.

Hardwired values for CIBR project team members (make sure you use the pre-configured values below):
* "function" should be set to "1" for CIBR runs (it specifies the FLARE function to run needs to be set to "1" (the automated deployment will sequence the execution of functions 2, 3, and 4 automatically)
* "use_https" should be set to "TRUE" for CIBR runs (it specifies whether HTTPS is used ("TRUE") or HTTP is used ("FALSE") for access to the S3 server)
* "aws_default_region" should be set to "s3" (it specifies the S3 server region)
* "aws_s3_endpoint" should be set to "flare-forecast.org" (it specifies the S3 server's name)

### Generate a pull request for your json file ##

#### Step1
Click on the button to fork the repo  
![](https://i.imgur.com/gmonWeV.png)

#### Step2
Make sure top-left corner shows that this repo is forked from original repo.
Click on Add file and Create new file buttons to create your own json file.
![](https://i.imgur.com/SqQuTKy.png)

#### Step3
Edit and Commit your own json file with specific format.
![](https://i.imgur.com/lOJq6cf.png)

![](https://i.imgur.com/QFYqDpo.png)

#### Step 4
After creating all json files, we are going to create the pull request to the repo.
Click on the pull request button from top menu. Then, click on the New Pull Request Button to create the pull request.
![](https://i.imgur.com/2V6FJ4K.png)

Make sure your repo and branch is correct.
Click on GREEN "create pull request" button.
![](https://i.imgur.com/lW3dDLR.png)
Add commit message if you want to. Then, click on GREEN "create pull request" button.
![](https://i.imgur.com/NSaFdtW.png)



If you see this page, it means that you have created the pull request. Once the moderator of the repo checks your json file, your pull request will be accepted and merged. Then, the ibmcloud will be deployed automatically.

![](https://i.imgur.com/YSB00nw.png)

## Retroactive Run Deployment

To run lake forecasts in a period in the past, follow these steps:

1- Fork the forecast code repository for the lake into your GitHub account. For instance, fork https://github.com/FLARE-forecast/FCRE-forecast-code.

2- Edit the following file in your newly forked repository and change the `sim_name`, `start_datetime` and `forecast_start_datetime` as you need. Different forecast projects on a single lake are distinguished by their `sim_name`. So, choose a unique name to avoid overwriting previous runs.
  - `configuration/default/configure_run.yml`

For instance:

```YAML
restart_file: .na
start_datetime: 2020-12-01 00:00:00
end_datetime: .na
forecast_start_datetime: 2020-12-02 00:00:00
forecast_horizon: 0
sim_name: fcre_2021
configure_flare: configure_flare.yml
configure_obs: observation_processing.yml
use_s3: TRUE
```

`start_datetime` is usually a day behind `forecast_start_daytime`. For a cold start without a `restart_file`, it is highly recommended to set the `forecast_start_datetime` earlier than the actual start date of the desired forecast period. The results for these early days are not as accurate as the rest of the forecast period and you can discard them later.

3- Now you can apply your desired changes to the rest of the codebase and tweak the forecast parameters in the configuration files.

4- Create a new JSON file, `fcre-2021.json`, for instance, and add the following content and modify it to reflect your desired retroactive forecast run. `number_of_runs` is the duration of the forecast period in days plus the cold start days.

```JSON
{
  "forecast_code": "https://github.com/vahid-dan/FCRE-forecast-code",
  "forecast_code_branch": "",
  "forecast_code_commit": "",
  "config_set": "default",
  "function": "1",
  "configure_run": "configure_run.yml",
  "use_https": "TRUE",
  "aws_default_region": "s3",
  "aws_s3_endpoint": "flare-forecast.org",
  "number_of_runs": 395
}
```

5- Send the JSON file to the CIBR team. A moderator will verify your JSON file and if everything is alright, the retroactive run should start shortly.

6- The outputs of the retroactive run will start to show up in the flare-forecast S3 storage as soon as they are generated. For example, for FCRE, the forecasts are located at `forecasts/fcre`. It may take a while before all the outputs for the whole specified period is ready.
