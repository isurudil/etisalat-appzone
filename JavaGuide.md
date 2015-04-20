# Getting Started: SMS Application > Java #

---


  * Software Requirements:
    * **JDK 1.6+**
    * **[Simulator](https://www.dropbox.com/s/5o38ufvecq4m0gp/sdk-standalone-1.1.50-distribution.zip)**
    * **Tomcat** (or any server of your choice)
    * **IDE of your choice** ( This guide is done on Eclipse IDE )



> This tutorial describes how to develop and deploy a simple SMS Application on the Aventura System using Java as the environment. The example application, will demonstrate how to receive users SMS and to respond them back with a message of your own.



## Creating a Project from Scratch ##

> If you’re using Eclipse as your IDE, go to **File -> New** and under **_‘Categories’_** type **‘Dynamic Web Project’** This is important, as you will be sending and receiving messages through HTTP.
![https://imagizer.imageshack.us/v2/613x500q90/841/5uf1.png](https://imagizer.imageshack.us/v2/613x500q90/841/5uf1.png)



> Then select **_‘next’_** . Now you have entered to the selecting **Servers** page. Select latest tomcat version as you will be deploying your project`s **WAR** file into this web container.

![https://imagizer.imageshack.us/v2/613x558q90/841/gc6u.png](https://imagizer.imageshack.us/v2/613x558q90/841/gc6u.png)

After the selection of the tomcat version, you should come across a window like this.

![https://imagizer.imageshack.us/v2/613x558q90/268/brw3.png](https://imagizer.imageshack.us/v2/613x558q90/268/brw3.png)


Once it is done, select **next** again. Now you have to name your project.

![https://imagizer.imageshack.us/v2/613x789q90/850/eihc.png](https://imagizer.imageshack.us/v2/613x789q90/850/eihc.png)

Click **Finish** button.

Once it is done you will be able to see your project in the left hand side of your IDE. Right click the **src** folder which you can find after expanding the project structure. Then select **New** > **Package**

Name it according to your preferred way.

![https://imagizer.imageshack.us/v2/613x445q90/809/1s04.png](https://imagizer.imageshack.us/v2/613x445q90/809/1s04.png)

Once it is done right click the package name, select **Class** and create a new Java Class. Name it according to your taste.

![https://imagizer.imageshack.us/v2/656x733q90/829/agz5.png](https://imagizer.imageshack.us/v2/656x733q90/829/agz5.png)

Now we are done with the creation of the project.


## Importing API’s ##

> The next step, is to attach the necessary API’s needed to create this java application. These can be downloaded by following links.

[Gson](https://www.dropbox.com/s/pj6ib9wed4o3hsw/gson-1.7.2.jar)

[SDP App API](https://www.dropbox.com/s/5fzgl9r67b3x5l9/sdp-app-api-1.1.50.jar)

[Servlet API](https://www.dropbox.com/s/ivxo7ruri1gvaz5/servlet-api-2.4.jar)

> After downloading, you can right-click on the project and select **Properties**. Then choose **java build path** and select **libraries**. Select **Add External JARs** and select the downloaded JARs and import them. Select **OK**.

![https://imagizer.imageshack.us/v2/661x520q90/849/kx7c.png](https://imagizer.imageshack.us/v2/661x520q90/849/kx7c.png)

> Now that all the necessary API’s are imported you can start implementing your idea.

## Implementing Your Idea ##

> For this tutorial, we will first create a java class named **_MessageReceiver_**. Then you will need to implement MoSmsListener interface. (You will need to code the following line in order to get this done.)
```
  public class MessageReceiver implements MoSmsListener {
```

> Afterwards you will have to override two methods by the name of **_init(ServletConfig servletConfig)_** and **_onReceivedSms(MoSmsReq moSmsReq)_**. Since all the messages that your application is going to receive will be passed onto this method, **_onReceivedSms(MoSmsReq moSmsReq)_**, it is important to note that your logic should also come inside this method.

For this tutorial, we will simply send a response message “Hello World !” back to the user.

> The following is how the **_MessageReceiver_** class would look like after completion.

```

package com.demo.application;

import hms.kite.samples.api.StatusCodes;
import hms.kite.samples.api.sms.MoSmsListener;
import hms.kite.samples.api.sms.SmsRequestSender;
import hms.kite.samples.api.sms.messages.MoSmsReq;
import hms.kite.samples.api.sms.messages.MtSmsReq;
import hms.kite.samples.api.sms.messages.MtSmsResp;

import javax.servlet.ServletConfig;
import java.net.URL;

public class MessageReceiver implements MoSmsListener {

	@Override
	public void init(ServletConfig servletConfig) {

	}

	@Override
	public void onReceivedSms(MoSmsReq moSmsReq) {
		try {
			System.out.println(moSmsReq);
			SmsRequestSender smsMtSender = new SmsRequestSender(new URL(
					"http://localhost:7000/sms/send"));

			MtSmsReq mtSmsReq;
			mtSmsReq = createSimpleMtSms(moSmsReq);

			mtSmsReq.setApplicationId(moSmsReq.getApplicationId());
			mtSmsReq.setPassword("d3d8c7fb7cd87659a6003e50fb1ba042");
			mtSmsReq.setSourceAddress("mykeyword");
			mtSmsReq.setVersion(moSmsReq.getVersion());

			MtSmsResp mtSmsResp = smsMtSender.sendSmsRequest(mtSmsReq);

			String statusCode = mtSmsResp.getStatusCode();
			String statusDetails = mtSmsResp.getStatusDetail();
			if (StatusCodes.SuccessK.equals(statusCode)) {
				System.out.println("MT SMS message successfully sent");
			} else {
				System.out
						.println("MT SMS message sending failed with status code ["
								+ statusCode + "] " + statusDetails);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	private MtSmsReq createSimpleMtSms(MoSmsReq moSmsReq) {
		MtSmsReq mtSmsReq = new MtSmsReq();
		MoSmsReq req = moSmsReq;
		String text = req.getMessage();
		mtSmsReq.setMessage("Hello World !");
		return mtSmsReq;
	}

}

```

## Defining and Mapping the Servlet Class ##

> In order to direct all the messages that comes to your application,into the **_MessageReceiver_** class’s implemented **_onReceivedSms_** method, you will need to define it as a Servlet Class and map it with a URL pattern.

> For this you will need to go inside the **_WEB-INF -> web.xml_** and make some modifications.

> The final output of the **web.xml** file would be as following.

```
<?xml version="1.0" encoding="ISO-8859-1"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
         version="2.4">

    <servlet>
        <servlet-name>moReceiver</servlet-name>
        <servlet-class>hms.kite.samples.api.sms.MoSmsReceiver</servlet-class>
        <init-param>
            <param-name>smsReceiver</param-name>
            <param-value>com.demo.application.MessageReceiver</param-value>
        </init-param>
    </servlet>

    <servlet-mapping>
        <servlet-name>moReceiver</servlet-name>
        <url-pattern>/mo-receiver</url-pattern>
    </servlet-mapping>    
</web-app>
```

## Building the Application ##

> _Right-Click_ the project and select **Export** > **WAR** and give the path to **Tomcat** > **Webapps**.


> Now you can deploy this on an application server of your choice. Refer [mChoice-Simulator guide](https://code.google.com/p/etisalat-appzone/wiki/SimulatorGuide) to test the application.


---

