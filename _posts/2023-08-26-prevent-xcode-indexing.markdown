---
layout: post
title:  "Prevent Xcode from Unnecessary Indexing"
date:   2023-08-26 13:55:13 +0300
categories: Xcode
---
## The Problem

In the realm of software development, an Integrated Development Environment (IDE) like Xcode offers a plethora of indispensable features – code auto-completion, syntax highlighting, refactoring capabilities, and the ability to swiftly navigate to definitions, among others. At the heart of these functionalities lies Xcode's indexing process, a mechanism through which the IDE parses and organizes files within a project. However, a challenge arises when a project accommodates files that, while crucial for runtime operations, do not bear any relevance to the source code.

Imagine a scenario where certain files in your project serve a purpose exclusively during runtime – they don't contribute to the source code but are indispensable nonetheless. This is where the complexity unravels. As the quantity and size of these runtime-only files escalate, so does the impact on the indexing process. The consequence? A discernible slowdown in Xcode's performance.

Let's proceed with exploring effective methods to alleviate these challenges and optimize your Xcode environment. My aim is to equip you with strategies for preventing unnecessary indexing, enhancing your development workflow.

## Sample

To shed light on this issue, I've prepared [a sample project](https://github.com/VMironiuk/xcode-indexing-samples/tree/main/issue/XcodeIndexingSample) that demonstrates this problem. Upon delving into the project, you'll immediately notice the conspicuous "Indexing Text" title, positioned prominently atop Xcode's interface, as visually depicted below:
![Xcode indexing](/assets/2023-08-26-prevent-xcode-indexing/xcode-indexing.png)

If you want to experience this issue for yourself, you can do a little experiment. By making copies of the files in a folder called `dictionaries`, you can make the indexing process even slower. I found something interesting when doing this experiment on my MacBook Pro from 2018 with an Intel processor. When the size of the `dictionaries` folder grew beyond 1 GB, my computer started to slow down a lot. This clearly shows how these additional files impact your computer's performance.

This situation emphasizes the need for solutions to avoid these slowdowns. It's important to find ways to make Xcode work smoothly, even when dealing with files that are only used during runtime. As we move forward in this blog post, we will discover effective methods to overcome these indexing challenges. This will ensure that your coding experience remains seamless and without interruptions. Let's continue and equip ourselves with the knowledge to overcome this challenge.

## Solution

Sadly, Xcode doesn't directly allow us to exclude certain files or folders from its indexing process. However, we can cleverly work around this limitation using custom scripts in Xcode.

The workaround involves a series of simple steps. The first step is to manually prevent certain files and folders from being indexed. We do this by compressing them into an archive. After that, we create a custom script in Xcode. This script is designed to go into a bundle's resource folder and unpack the archive.

### Add a Custom Script

To add a custom script, (1) choose your project in the Project navigator, (2) select a target from the Target list, (3) navigate to the Build Phases tab, (4) click the Plus button at the top left corner, (5) and choose the New Run Script Phase menu item.
![Create a custom script in Xcode](/assets/2023-08-26-prevent-xcode-indexing/create-custom-script.png)

### Give the Script a Meaningful Name

Not a mandatory step, though, but it can be helpful for your teammates or your future self. To rename the script phase, double-click the Run Script title to edit it. For instance, I named mine `Unzip Dictionaries`.
![Giving the Script a Meaningful Name in Xcode](/assets/2023-08-26-prevent-xcode-indexing/rename-custom-script.png)

### Writing Script

Click the arrow button to expand the newely added script phase. You can see a text editor to write your script. Let's put our script here:
![Expanded newely created custom script in Scode](/assets/2023-08-26-prevent-xcode-indexing/new-custom-script-expanded.png)

#### 1. Specify a path to our archive. 
{% highlight bash %}
DICTIONARIES_ZIP_PATH="${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}"
{% endhighlight %}
Here, `BUILT_PRODUCTS_DIR` - dentifies the directory under which all the product’s files can be found and 
`UNLOCALIZED_RESOURCES_FOLDER_PATH` - specifies the directory that contains the product’s unlocalized resources.

#### 2. Specify the archive name.
{% highlight bash %}
DICTIONARIES_ZIP="dictionaries.zip"
{% endhighlight %}

#### 3. Incorporate the process of unpacking the archive into our script.
{% highlight bash %}
if [ -e "${DICTIONARIES_ZIP_PATH}/${DICTIONARIES_ZIP}" ]; then
    cd "${DICTIONARIES_ZIP_PATH}"
    unzip "$DICTIONARIES_ZIP"
    rm "$DICTIONARIES_ZIP"
    cd -
fi
{% endhighlight %}

In this sequence, the script verifies the archive's existence. If confirmed, it navigates to the bundle's resource folder, unpacks the archive, removes the archive to keep things tidy, and then retreats back to its previous location.

## Bonus Improvement

Another exciting improvement awaits – one that optimizes how the script runs when you build your project. Instead of running the script every single time we build the project, wouldn't it be great if Xcode only runs the script when there are changes to the `dictionaries.zip` archive? Luckily, Xcode offers a way to achieve this through `Input and Output Files`.

The concept behind `Input and Output Files` is quite simple. Xcode keeps an eye on these files. When they change, Xcode triggers the script during the build process. If there are no changes, the script doesn't run. This smart feature can greatly enhance our script execution much smarter.

To add files to your script, click the Add button (+) in the appropriate section of your Run Script build phase. For each entry, specify the path to the file.
![Input/Output Files in Xcode](/assets/2023-08-26-prevent-xcode-indexing/IO-files.png)

For `Input Files` enter the path:
{% highlight bash %}
$(SRCROOT)/dictionaries.zip
{% endhighlight %}

For `Output Files` set next path:
{% highlight bash %}
$(DERIVED_FILE_DIR)/unzip-dictionaries-output.txt
{% endhighlight %}

And last but not least, we actually need to create `unzip-dictionaries-output.txt` at the end of our script execution. Add this line to the end of the script:
{% highlight bash %}
echo "SUCCESS" > "${DERIVED_FILE_DIR}/unzip-dictionaries-output.txt"
{% endhighlight %}

With this final change, the script will look like the one below:
{% highlight bash %}
DICTIONARIES_ZIP_PATH="${BUILT_PRODUCTS_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}"
DICTIONARIES_ZIP="dictionaries.zip"

if [ -e "${DICTIONARIES_ZIP_PATH}/${DICTIONARIES_ZIP}" ]; then
    cd "${DICTIONARIES_ZIP_PATH}"
    unzip "$DICTIONARIES_ZIP"
    rm "$DICTIONARIES_ZIP"
    cd -
fi

echo "SUCCESS" > "${DERIVED_FILE_DIR}/unzip-dictionaries-output.txt"
{% endhighlight %}

With this, you're well-prepared to manage the intricacies of runtime files and optimize your Xcode experience. Feel free to dive into the completed project and experiment with the workaround yourself. You can find the full project, along with its solution, right [here](https://github.com/VMironiuk/xcode-indexing-samples/tree/main/fixed/XcodeIndexingSample).

The screenshot below shows up the completed version of the custom script:
![Custom script completed](/assets/2023-08-26-prevent-xcode-indexing/custom-script-completed.png)

# Resources

[Running custom scripts during a build](https://developer.apple.com/documentation/xcode/running-custom-scripts-during-a-build?language=objc)

[Declare inputs and outputs for custom scripts and build rules](https://developer.apple.com/documentation/Xcode/improving-the-speed-of-incremental-builds#Declare-inputs-and-outputs-for-custom-scripts-and-build-rules)

[The sample project repository](https://github.com/VMironiuk/xcode-indexing-samples)