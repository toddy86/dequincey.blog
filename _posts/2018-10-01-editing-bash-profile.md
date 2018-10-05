---
layout: post
title: Editing a Bash Profile on a Mac
categories: tutorial
---

A step-by-step guide on how to access and edit the bash profile on a Mac.

<!-- more -->
<b>Project goal</b> 

Learn how to edit a bash profile and document the process as a tutorial, as tutorials on this topic do not appear to be very plentiful.

Steps 1 to 2 relate to identifying the available Java versions. Updating the bash profile starts at step 3.

---
<b>Lessons learnt</b>
<ul>
  <li>How to access and edit a bash profile</li>
  <li>How to update the default path for Java</li>
</ul>

---
<h3>Step 1: Identify Current Default Java Version</h3>

{% highlight bash %}
java -version
{% endhighlight %}

---
<h3>Step 2: Identify All Available Versions of Java</h3>

{% highlight bash %}
/usr/libexec/java_home -V
{% endhighlight %}

---
<h3>Step 3: Access Bash Profile</h3>

{% highlight bash %}
touch .bash_profile
open -a TextEdit.app .bash_profile
{% endhighlight %}
The above will open the bash profile with the Mac TextEdit app.

---
<h3>Step 4: Editing Bash Profile</h3>

In the opened text document, add the required export path for Java as follows.

{% highlight bash %}
export JAVA_HOME=`/usr/libexec/java_home -v 1.8.0_05`
{% endhighlight %}

---
<h3>Step 5: Confirm the Updates</h3>

{% highlight bash %}
java -version
{% endhighlight %}
The above should return the version of Java from the newly assigned Java home path.