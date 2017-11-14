- [Compelld](#compelld)
    + [CI](#ci)
      - [Develop branch:](#develop-branch-)
      - [Master branch:](#master-branch-)
    + [CD](#cd)
    + [Creation of Superuser](#creation-of-superuser)

# Compelld 

| Component | Public IP |
| ------ | ------ |
| Jenkins | 13.57.7.59 |
| Docker | 54.219.162.29 |

Login to Jenkins UI by hitting the Public IP in browser.

### CI
#### Develop branch: #### 
1. To execute the CI pipeline for compelld build the following pipeline:
[CI_Compelld_dev]. To execute the job in the Jenkins dashboard click on the job mentioned above. Then on the upper left panel click on “Build now”. This will auto-trigger [CI_Compelld_build_dev]. This will execute the makemigration and migrate on the application.
2. This will auto-trigger another job: [CI_Compelld_UT_dev]. This will execute the test cases and generate coverage report. 
3. The coverage report can be viewed by clicking on the job [CI_Compelld_UT_dev] from jenkins dashboard then clicking on "Coverage Report" section on the left panel of the job.
4. The [CI_Compelld_build_dev] will check for changes every 5 minutes, and will auto trigger if there is change in code. 
5. Alternatively to execute the pipeline you can also build the following job directly: [CI_Compelld_build_dev]. This will also auto-trigger [CI_Compelld_UT_dev]
6. The output of each job can be checked on the Console output of the build on the lower left panel of the job dashboard.

#### Master branch: #### 
1. To execute the CI pipeline for compelld build the following pipeline:
[CI_Compelld_master]. To execute the job in the Jenkins dashboard click on the job mentioned above. Then on the upper left panel click on “Build now”. This will auto-trigger [CI_Compelld_build_master]. This will execute the makemigration and migrate on the application.
2. This will auto-trigger another job: [CI_Compelld_UT_master]. This will execute the test cases and generate coverage report. 
3. The coverage report can be viewed by clicking on the job [CI_Compelld_UT_master] from jenkins dashboard then clicking on "Coverage Report" section on the left panel of the job.
4. The [CI_Compelld_build_master] will check for changes every 5 minutes, and will auto trigger if there is change in code. 
5. Alternatively to execute the pipeline you can also build the following job directly: [CI_Compelld_build_master]. This will also auto-trigger [CI_Compelld_UT_master]
6. The output of each job can be checked on the Console output of the build on the lower left panel of the job dashboard.

### CD
1. To deploy a new docker container execute the following job: [CD_Compelld_build]
This job will ask for 2 input parameters:
    1. The branch you want to deploy the code from.
    2. The {public IP/hostname}:{port} of the instance the application will be          running on, which is the docker host.  The value you provide here will be pushed     to config.ini as a Host parameter. Ex: 54.219.162.29:8000 The port will always      be 8000
2. This job will pull the code from the branch specified above and execute makemigration and migrate.
3. This will auto trigger another job: [CD_Compelld_UT]. This job will execute the test cases and generate coverage report.
4. This job will auto trigger another job: [CD_compelld_deploy_docker_postgres]
Which will deploy the postgres container. The postgres version will be latest. It will create a user with a password mentioned in the same job as parameters (PostGresUser, PostGresPassword).
The username and password mentioned here will be added to config.ini in the Django container as well. 
5. The postgresql data directory is mounted at **/var/lib/postgres/data** in docker host. So it will remain persistent across container respin. To clean the database data you will have to delete **/var/lib/postgres/data** in docker host. The jenkins job will create the new data directory and create all the necessary files as well. The compellddb database will be created if not present. Postgresql will be running on 5432 on docker host.
6. This will autotrigger [CD_compelld_deploy_docker_Django_UT]. This job will create the django container which will have nginx and django application running. The nginx will proxy pass the django application running on port 8000 on container to port 80 on docker host. 
7. The config.ini will be edited runtime by the jenkins job via the script **config.sh**
If you want to add any dynamic parameter in config.ini via jenkins job just add the variable in the below line in config.sh: declare -a properties=("Host" "PostGresHost" "PostGresUser" "PostGresPassword" "logfile_path")
Here the new variable is **logfile_path**.

8. In the jenkins job [CD_compelld_deploy_docker_Django_UT] you can pass the parameter to the script in the shell script as below:
```sh
/bin/bash ~/compelld-deployment/config.sh $Host $PostGresHost $PostGresUser $PostGresPassword $logfile_path
```
9. The config.sh will generate config_new.ini and copy it into the django container as /opt/config.ini

10. The same job will then activate the django_python3.6 virtualenv we created in image creation step. Then it will execute the makemigrations and migrate. Then it will execute loaddata and then execute the test cases inside the container.
11. This job will autotrigger: [CD_compelld_deploy_docker_django]. This job will start the django application. The job will also start nginx service inside the container.
12. After the job is successful you can access the application on the public ip of the docker host on port 80. The UI can be accessed as {public IP of docker host}/admin and API as {public IP of docker host}/compelld/v1. 

### Creation of Superuser
1. To login into the compelld admin UI you will need to create superuser.
2. Execute the following commands for it:
    1. SSH into the docker host
    2. login to user compelld-docker:
    
    ```sh
    sudo su compelld-docker
    ```
    3. Execute the following command:
    ```sh
    docker exec -ti compelld_django /bin/bash
    ```
    You will be logged into compelld_django container.
    4. Then execute the following command to start python virtualenv:
    ```sh
    source /django_python_3.6/bin/activate
    ```
    5. Then execute the following to create superuser:
    ```sh
    python /opt/k-apid/compelld/manage.py createsuperuser
    ```
    You will be asked for a email and password. Enter email, press <enter> key. Enter password , press <enter> key. Confirm password.
    6. Then exit the container by executing:
    ```sh
    exit
    ```
    7. Superuser will be created. Now in browser enter {public IP of docker host}/admin It will ask for login, enter the email and password added in superuser command. you shall be logged into compelld admin console.

[CI_Compelld_build_dev]: <http://13.57.7.59/job/CI_Compelld_build_dev/>
[CI_Compelld_UT_dev]: <http://13.57.7.59/job/CI_Compelld_UT_dev//>
[CI_Compelld_dev]: <http://13.57.7.59/job/CI_Compelld_dev/>
[CI_Compelld_master]: <http://13.57.7.59/job/CI_Compelld_master/>
[CI_Compelld_build_master]: <http://13.57.7.59/job/CI_Compelld_build_master/>
[CI_Compelld_UT_master]: <http://13.57.7.59/job/CI_Compelld_UT_master/>
[CD_Compelld_build]: <http://13.57.7.59/job/CD_Compelld_build/>
[CD_Compelld_UT]: <http://13.57.7.59/job/CD_Compelld_UT/>
[CD_compelld_deploy_docker_postgres]: <http://13.57.7.59/job/CD_compelld_deploy_docker_postgres/>
[CD_compelld_deploy_docker_Django_UT]: <http://13.57.7.59/job/CD_compelld_deploy_docker_Django_UT/>
[CD_compelld_deploy_docker_django]: <http://13.57.7.59/job/CD_compelld_deploy_docker_django/>
