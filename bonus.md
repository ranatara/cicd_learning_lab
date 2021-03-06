**_Before beginning this step, be sure to be at a command line prompt from your prepared working environment.  This will either be your local machine, or within the provided container._**

#### Reminder: Working in the docker container

```
# Start a clean instance of the container
docker run -it --name cicdlab hpreston/devbox:cicdlab

[root@cf95a414877e coding]# exit

# If you need to restart an exited container
# Verify that you have  a container in a stopped state
docker ps -a

CONTAINER ID        IMAGE                         COMMAND             CREATED             STATUS                        PORTS               NAMES
cf95a414877e        hpreston/devbox:cicdlab       "/bin/bash"         2 minutes ago       Exited (0) 10 seconds ago                         cicdlab

# Restart your stopped container
docker start -i cicdlab

[root@cf95a414877e coding]#
```

[item]: # (slide)

## Let's put that pipeline to use!

[item]: # (/slide)

Now that we have our full CICD pipeline up and running, let's make some application changes and watch them deploy.

[item]: # (slide)

## Verify that the default app status

[item]: # (/slide)

The default demo app has a single route `hello/world`.  We will be adding an additional route to the application, but let's verify that it doesn't exist already.

1. Try to navigate to `hello/universe` at your application.  This would be available at a URL such as `http://class-USERNAME.mantl.domain.com/hello/universe`, but replace **USERNAME** with your Docker Username, and **mantl.domain.com** with the Lab Application Domain provided by your instructor.
2. You should get back a page not found error.

[item]: # (slide)

    ![App Hello Universe Error](images/app_hello_universe_error.png)

[item]: # (/slide)

[item]: # (slide)

## Add /hello/universe to the app

[item]: # (/slide)

**_In this step you will be entering several commands in a terminal window.  These need to be run from your local repo directory.  If you followed the directions when cloning the repo locally, this command will place you in the correct directory_**

```
cd ~/coding/cicd_demoapp
```

## Build the Test

1. In your editor or IDE, open `testing.py` and add the new tests for /hello/universe.  You can simply copy and paste the below into your editor.

[item]: # (slide)

```
import demoapp
import unittest


class FlaskTestCase(unittest.TestCase):

    def setUp(self):
        demoapp.app.config['TESTING'] = True
        self.app = demoapp.app.test_client()

    def test_correct_http_response(self):
        resp = self.app.get('/hello/world')
        self.assertEquals(resp.status_code, 200)

    def test_correct_content(self):
        resp = self.app.get('/hello/world')
        self.assertEquals(resp.data, '"Hello World!"\n')

    def test_universe_correct_http_response(self):
        resp = self.app.get('/hello/universe')
        self.assertEquals(resp.status_code, 200)

    def test_universe_correct_content(self):
        resp = self.app.get('/hello/universe')
        self.assertEquals(resp.data, '"Hello Universe!"\n')

    def tearDown(self):
        pass

if __name__ == '__main__':
    unittest.main()

```

[item]: # (/slide)

2. Since we are **NOT** changing the actual build process stored in `.drone.yml`, we don't need to recreate the secrets file.  Once the build process is complete, as long as new tests or steps aren't added these files can be left alone.  Developers can focus on their code and changes, not the build process.
3. Commit and push our updated test to begin the CICD process.

[item]: # (slide)

    ```
    # add the file to the git repo
    git add testing.py

    # commit the change
    git commit -m "Added new Tests for hello/universe"

    # push changes to GitHub
    git push
    ```

[item]: # (/slide)

4. Check the drone server to check the status of the build.  You can also check in Spark for Status updates.  

[item]: # (slide)

    ![Drone Build](images/drone_5-5_build.png)

[item]: # (/slide)

5. The build failed because we built the test **BEFORE** the feature.  That's test driven development, and the failure is exactly what we want to see.  When drone fails in the testing, the new container isn't built and published.  

### Build the Feature

1. In your editor or IDE, open `demoapp.py` and add the new class and resource for HelloUniverse.  You can simply copy and paste the below into your editor.

[item]: # (slide)

```
from flask import Flask, request
from flask_restful import Resource, Api, reqparse


app = Flask(__name__)
app = Flask(__name__)
api = Api(app)

class HelloWorld(Resource):
    def get(self):
        text = "Hello World!"
        return text

api.add_resource(HelloWorld, '/hello/world')

class HelloUniverse(Resource):
    def get(self):
        text = "Hello Universe!"
        return text

api.add_resource(HelloUniverse, '/hello/universe')

if __name__ == '__main__':
    # Run Flask
    app.run(debug=True, host='0.0.0.0', port=int("5000"))

```

[item]: # (/slide)
    
2. Since we are **NOT** changing the actual build process stored in `.drone.yml`, we don't need to recreate the secrets file.  Once the build process is complete, as long as new tests or steps aren't added these files can be left alone.  Developers can focus on their code and changes, not the build process.
3. Commit and push our updated application to begin the CICD process.

[item]: # (slide)

    ```
    # add the file to the git repo
    git add demoapp.py

    # commit the change
    git commit -m "Added hello/universe to the application"

    # push changes to GitHub
    git push
    ```

[item]: # (/slide)

4. Check the drone server to verify the build has begun.  You can also monitor the Spark room for the completed message.

[item]: # (slide)

    ![Drone Build](images/drone_6th_build.png)

[item]: # (/slide)

5. Once drone completes the build, check Marathon and watch as the application restarts.

[item]: # (slide)

    ![Marathon App Restart](images/marathon_app_restart.png)

[item]: # (/slide)

6. Wait for Marathon to show the application as healthy.

[item]: # (slide)

    ![Marathon App Healthy](images/marathon_app_healthy.png)

[item]: # (/slide)

7. Refresh the web page with the `hello/universe` page **Not Found**.  You should now have your new message available.

[item]: # (slide)

    ![App Hello Universe](images/app_hello_universe.png)

[item]: # (/slide)

8. Feel free to experiment with more changes to the application.  Each commit will result in the application restarting with the updated code.

[item]: # (slide)

## Current Build Pipeline Status

![Final Diagram](images/stage_final_diagram.png)

[item]: # (/slide)

Okay, so let's review the steps in the full pipeline.

1. You committed and pushed code to GitHub.com
2. GitHub sent a WebHook to the Drone server notifying it of the committed code.
3. Drone checks the _.drone.yml_ file and executes the commands in the _build_ phase. During this phase, Drone has two steps: 
	* *build_starting* 	
  		* Fetch a container from hub.docker.com to begin the build.  This container is identified in the `image: python:2` line of the drone config file.  Drone will run the commands listed in this section
	* *run_tests*
  		* Fetch a container from hub.docker.com to do the testing.  This container is identified in the `image: python:2-alpine` line of the drone config file.  Drone will run the tests described by the commands listed in this section

4. Drone checks the _.drone.yml_ file and executes the commands in the _publish_ phase. During this phase, Drone will: 
  * Build a Docker Container using the Dockerfile definition included in the Git repo
  * Push the container up to hub.docker.com using the credentials contained in the secrets file
5. Drone checks the _.drone.yml_ file and executes the commands in the _deploy_ phase. In this phase, the following actions will take place: 
  * Drone sends a WebHook command to Marathon to cause an application restart
  * Marathon pulls the new container from hub.docker.com containing the code changes
6. Drone checks the _.drone.yml_ file and executes the commands in the _notify_ phase. During notification, Drone will check the staus of the build and then send notifications to the Spark room.
  * If the build was successful, send a Success notification
  * If the build failed, send a Failure notificiation and blame someone.

[item]: # (slide)

## Next Step!

8. [Clean-Up](cleanup.md)

[item]: # (/slide)

Time to move onto the final step, clean-up!


