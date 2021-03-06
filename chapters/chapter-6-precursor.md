# Precursor: Login and Accessing with Postman

## Overview

Using Adobe AdminConsole to manage an Experience Platform instance, manage Product Profiles, manage users and administrators. Validate if login to Experience Platform UI succeeds.

## Learning Objectives

- Understand AdminConsole capabilities for Experience Platform
- Setting up an Adobe I/O Integration
- Create your local websever to host a WeTravel demo site for data streaming (Chapter 9)

## Lab Resources

- Adobe I/O Console: [https://console.adobe.io/](https://console.adobe.io/)

## Lab Tasks

- Setup an integration
- Authenticate via POSTMan
- Install the Fenix Web Server with WeTravel

### Setup an integration

1. Create Certificate

   **For MacOS & Linux platform**

   Open terminal and execute below command:

   ```shell
   openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out certificate_pub.crt
   ```

   **For Windows Platform**

   For Windows users, please make sure you have **OpenSSL** set up

    - Download an OpenSSL client [OpenSSL](https://bintray.com/vszakats/generic/download_file?file_path=openssl-1.1.1-win64-mingw.zip)

   Extract this zip file to the directory `C:\libs`


   Open Command Line Prompt and execute below commands.

   ```shell
   set OPENSSL_CONF=C:/libs/openssl-1.1.1-win64-mingw/openssl.cnf
   cd C:/libs/openssl-1.1.1-win64-mingw/
   openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout private.key -out certificate_pub.crt
   ```

   You will get a response similar to the following which prompts you to enter some information about yourself:

   ```
   Generating a 2048 bit RSA private key
   .................+++
   .......................................+++
   writing new private key to 'private.key'
   -----
   You are about to be asked to enter information that will be incorporated
   into your certificate request.
   What you are about to enter is what is called a Distinguished Name or a DN.
   There are quite a few fields but you can leave some blank
   For some fields there will be a default value,
   If you enter '.', the field will be left blank.
   -----
   Country Name (2 letter code) []:
   State or Province Name (full name) []:
   Locality Name (eg, city) []:
   Organization Name (eg, company) []:
   Organizational Unit Name (eg, section) []:
   Common Name (eg, fully qualified host name) []:
   Email Address []:
   ```

   After entering the information two files will be generated: `certificate_pub.crt` and `private.key`. These files can be found in the same directory that you ran the `openssl` command from.

   Note `certificate_pub.crt` will expire in 365 days. You can make the period longer by changing the value of days in the openssl command above but rotating credentials periodically is a good security practice.

   The `certificate_pub.crt` certificate will later be uploaded to the Adobe IO Console for when you create an API key for access to any Adobe I/O API.

   Your private key file named `private.key` will be used later to sign your JWT token.

   > Note: Don't close this terminal window as you will need it later.

1. Navigate to the [Adobe I/O Console](https://console.adobe.io/) and sign in with your Adobe ID.
1. From this page we want to create a New Integration.

   ![](../images/chapter-2/new_integration.png)

1. You will then be prompted to Access an API or to Receive near-real-time events. We will be accessing APIs so select Access an API and then Continue.

   ![](../images/chapter-2/new_integration1.png)

1. The drop-down menu on the top right of the screen is where you would switch your organization if your account is tied to multiple. We are selecting **Workshop** and Data Services under Experience Cloud since we want to access the data services.

   ![](../images/chapter-2/new_integration2.png)

1. After your organization is selected there will be a new prompt at the top. We want a New Integration so make sure that option is selected before clicking Continue

   ![](../images/chapter-2/new_integration3.png)

1. Fill in your Integration Details. Afterwards, click on Select a File to upload your certificate_pub.crt file we generated in the previous section. Click Create Integration to finish up the process

   ![](../images/chapter-2/new_integration4.png)

1. After creating your integration, you will be able to view the details of your integration. After clicking on Retrieve client Secret your screen should look similar to this.

   ![](../images/chapter-2/access_values.png)

   Copy down the values for {API KEY}, {IMS ORG} which is the Organization ID, and {CLIENT SECRET} as these will be used in the next step.

### Authenticate via POSTMan

1. Start POSTMan

   ![](../images/chapter-2/postman.png)

1. Click the `Import` button on the top left.

   ![](../images/chapter-2/postman_import.png)

   Select the [PlatformSummit.postman_collection.json](../postman/PlatformSummit.postman_collection.json) collection file from this repository.

1. Next we need to import our environment. Click on the settings logo

   ![](../images/chapter-2/postman_settings.png)

   To bring up the **Manage Environments** dialog.

   ![](../images/chapter-2/postman_manage_env.png)

1. Then click on `Import`

   ![](../images/chapter-2/postman_import_env.png)

1. Select the [PlatformSummit.postman_environment.json](../postman/PlatformSummit.postman_environment.json) file to import the environment.

   ![](../images/chapter-2/postman_after_env_import.png)

1. Now click on the newly imported `Platform Summit - Environment`.

   ![](../images/chapter-2/postman_set_env.png)

1. Fill out the values for:

   - clientID
   - clientSecret
   - OrgID
   - TechAcctID

   that you generated when you created your **new integration**.

   **Fill these out in both the "Initial Valeus" and the "Current Values" field.**

   Also fill out the `ldap` field with your user id (first initial + last name) so you'll be able to uniquely identify the datasets you create.

1. Copy the contents of the `private.key` and use it as the value for `secret`.

   **For MacOS & Linux platform**

   From the same terminal you ran `openssl`, execute the following command:

   ```shell
   pbcopy < private.key
   ```

   **For Windows Platform**

   From the same terminal you ran `openssl`, execute the following command:

   ```shell
   notepad private.key
   ```

   Copy the entire key to the keyboard, including the `-----BEGIN PRIVATE KEY-----` and `-----END PRIVATE KEY-----` lines up to the last `-`.

1. Click `Update` and close the `Manage Environments` dialog.

1. Now make sure you select the `Platform Summit - Environment` from the environments drop down at the top right of POSTMan.

   ![](../images/chapter-2/postman_experience_platform_env.png)

1. After all this setup you are now ready to generate an JWT and bearer token to interact with Adobe I/O. In order to make this process easier we'll be using an Adobe I/O Runtime action.

   From our newly imported `Platform Summit` collection, open `Pre-Chapter 6` and click on `Adobe I/O Runtime: Generate Auth`. Then click on the body tab:

   ![](../images/chapter-2/postman_auth_body.png)

   All of that work you did to setup the environment has been put to good use. Each POSTMan call will take advantage of these values.

1. Now click `Send` and scroll down to the response section:

   ![](../images/chapter-2/postman_auth_response.png)

   That JSON response includes an `access_token` which is the Bearer token used to authenticate with Adobe I/O. The POSTMan call will save this value in an environment variable for future use.

---

### Deploying WeTravel webpage to your machine via Fenix webserver

1.  Download the [WeTravel website](../data/WeTravel-local.zip).
1.  Unzip the `WeTravel-local.zip` file and make note of which directory you extract the content to.
1.  Start the Fenix web server.

    ![](../images/chapter-9/start-fenix.png)

1.  Select the `Web Servers` menu and click `New`

    ![](../images/chapter-9/new-server.png)

1.  Use `WeTravel` as descriptive name.

    ![](../images/chapter-9/we-travel.png)

1.  Click on the folder in the `Directory` input field. Then navigate the file dialog to the directory you unzipped the `WeTravel-local` folder, highlight it and click `Select`.

    ![](../images/chapter-9/select-folder.png)

1.  Click `Create`.

    ![](../images/chapter-9/create.png)

1.  Click on the `Play` button to start the server.

    ![](../images/chapter-9/click-start.png)

1.  Now your WeTravel site should be ready to be browsed.

    ![](../images/chapter-9/server-started.png)

1.  Navigate to [http://127.0.0.1](http://127.0.0.1) to test the web server. You should see the following:

    ![](../images/chapter-9/localhost.png)

1.  Now we need to redirect requests to `we-travel.com` to our local web server.

    For **MacOS** users:

    - Open the Terminal program.
    - Run the following command:

              sudo nano /etc/hosts

      - You will need to type in your adminstrator password to continue.

      ![](../images/chapter-9/sudo-nano.png)

      - You're now in the Nano text editor. You should see something that looks like this:
      - Use the arrow keys to navigate the cursor to the line:

              127.0.0.1  localhost

      ![](../images/chapter-9/localhost-line.png)

      - Add `we-travel.com` to the end of the line so it looks like:

              127.0.0.1  localhost   we-travel.com

      ![](../images/chapter-9/we-travel-line.png)

      - Once you're done, hold down the control and O keys to save the file, then control and X to exit.
      - Finally you'll need to restart your Mac's DNS responder by entering the following command into the Terminal:

              sudo killall -HUP mDNSResponder

    For **Windows** users:

    - Press the Windows key.
    - Type **Code** in the search field to find the Code Editor you downloaded (I.e. VS Code).
    - In the search results, right-click **Code** and select **Run as Administrator**.
      From **Code**, open the following file:

            c:\Windows\System32\Drivers\etc\hosts

    - Add a new line to the end of the file that looks like this:

            127.0.0.1   we-travel.com

    - Click File > Save to save your changes.

1.  Navigate to [http://we-travel.com](http://we-travel.com) to test the web server. You should see the following:

    ![](../images/chapter-9/not-localhost.png)

---

#### Chapter Wrap

Whew! We are finally ready to start calling the Adobe Experience Platform API's for real. We've run through creating an integration and getting authenticated.

#### Additional Resources

[Authentication Documentation on Adobe I/O](https://www.adobe.io/authentication.html)

---

### Navigate

| **Previous:**                                                                          | **Next:**                                                               |
| -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Chapter 5 - [UI - Segmentation: Building Segments from Unified Profiles](chapter-5.md) | Chapter 6 - [API - Schema: Explore and Define XDM Schema](chapter-6.md) |

**Return Home:** [Workbook Index](../README.md)
