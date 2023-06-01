---
title: "A Simpler Entry to DevOps"
seoTitle: "A Simpler Entry To DevOps"
datePublished: Tue Oct 11 2022 14:51:39 GMT+0000 (Coordinated Universal Time)
cuid: cl94bmyzy000g09l16dded4qq
slug: a-simpler-entry-to-devops
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1665472859882/hGXVEFmrR.webp
tags: developer, automation, devops, infrastructure-as-code, automation-testing

---

### \*For the Beginners By a Beginner \*

As the heading suggests this blog is going to be for a fresher or a beginner who has just started his/her DevOps journey . Going forward we'll get to understand the basics of DevOps such as What exactly DevOps is ? Why DevOps is important? what role DevOps plays in the Software Development Life Cycle **(SDLC)** and how can we make the best use of it in subsequent blogs. All the pro people out there, who will be going through this blog please suggest what else we should focus on to be a Pro like you in the world of DevOps. Without a further due let's jump to our first focus area :

### What is DevOps?

Some say DevOps is a blending of Development and Operations, some say it's just the Operation, some says is a procedure of working in SDLC, some say it is the SDLC and so on. There is no such perfect definition available for DevOps but as per understanding from several sources we can define DevOps as a \*"Cultural practice across the organizations by development team and operation team to combine make use of the tools and technologies to fasten the product delivery without compromising the quality of the product" \* DevOps is complementary with Agile software development; several DevOps aspects came from the Agile way of working.

**What is the DevOps lifecycle?**

DevOps is another coating above the Agile way of development, as we all know agile methodology is famous for its agility which means faster delivery, less documentation, close work bond among the Development and Testing teams and many more. Whereas DevOps brings another aspect closer to Dev and Qa which is IT Operations and Infrastructures. DevOps personnel are the team that takes care of the entire infrastructure required for a smooth transition of an idea to a working product. The DevOps approach embraces continuous innovation, agility, and scalability to build, test, consume, and evolve software products. It promotes a culture of experimentation, feedback, and constant learning to reinvent products, services, and processes. However, to implement DevOps, a proper understanding of different phases of the DevOps lifecycle is crucial.

\*\*There are three ways of achieving DevOps which are: \*\*

* ***Flow thinking :*** DevOps flow thinking refers to the mind set that achieves continuous momentum using the blended functional pipeline activity performed by a dynamic team.
    
* ***Amplify Feedback :*** It focuses on the continuous feedback over the developed portions
    
* ***Experiment and learn :*** It involves the experiment of suitable tools and technologies and learning them to yield the best out of them.
    

### Components of DevOps

There are several key components of DevOps and here we will be discussing few of them

1.**Continuous development:**

This component involves the Designing and Coding part of the product here, the entire development process gets broken down into smaller development cycles. This phase is instrumental in mapping the vision for the entire development cycle, enabling developers to fully understand project expectations. Through this, the team starts visualizing its end goal as well. There are no DevOps tools required for planning, but many version control tools such as Git , Mercurial, and SVN. are used to maintain code. This process of code maintenance is called source code maintenance. Popular tools for source code maintenance include JIRA, Git, Moreover, there are different tools for packaging the codes into executable files, such as Ant, Gradle, and Maven. These executable files are then forwarded to the next component of the DevOps lifecycle.

**2\. Continuous integration:**

Continuous integration (CI) includes different steps related to the execution of the test process. In the modern way of agile development we are asked to develop the product in several phases rather than delivering the product at once , that's where CI plays vital role. CI becomes the hub for resolving these frequent changes on a daily or monthly basis. Building code is a combination of unit and integration testing, code review, and packaging. Since developers make frequent changes, they can quickly spot problems (if any) and resolve them at an early stage.

**3\. Continuous testing:**

Next in the DevOps lifecycle is the testing phase, wherein the developed code is tested for bugs and errors that may have made their way into the code. This is where quality analysis (QA) plays a major role in checking the usability of the developed software. Automation tools, such as JUnit, Selenium, cypress, apium and TestNG, are used for continuous testing, enabling the QA team to analyze multiple code-bases simultaneously. Doing this ensures that there are no flaws in the functionality of the developed software. Automated testing is done on automation tools like Selenium, after which the reports are generated on another automation tool, for example, TestNG. Automation of the entire testing phase also becomes possible with the help of the continuous integration tool Jenkins. Automation testing plays a vital role in saving time, labor and effort.

**4\. Continuous deployment:**

Continuous deployment (CD) ensures hassle-free product deployment without affecting the application’s performance. It is necessary to ensure that the code is deployed precisely on all available servers during this phase. This process eliminates the need for scheduled releases and accelerates the feedback mechanism, allowing developers to address issues more quickly and with greater accuracy.

**5\. Continuous monitoring:**

This phase processes important information about the developed app. Through continuous monitoring, developers can identify general patterns and gray areas in the app where more effort is required. Continuous monitoring is an operational phase where the objective is to enhance the overall efficiency of the software application. Moreover, it monitors the performance of the app as well. Therefore, it is one of the most crucial phases of the DevOps lifecycle. Different system errors such as ‘server not reachable’, ‘low memory’, etc., are resolved in the continuous monitoring phase.

# CAMS The Core Values of DevOps

### C : for Culture

Culture is the most important part of the DevOps movement. But what are we talking about here? Culture will bring a whole set of practices:

the use of Scrum, the presence of silos, the will-ness to limit the level of technical debt, the organization of retrospectives, … However, be careful : don’t try to replicate practices of other organizations as they are, only for the reason that it works in these organizations. This is called the Cargo Culting effect. You must first examine the problems that these other organizations have tried to solve and see how they solve them. Only after you can determine what will work for your own organization.

### A : Automation

Somewhere I read "Anything that can be automated should be Automated !"

So why automate? Above all to speed up the flow of information and avoid recruiting people.

Speeding up the flow of information, of course, brings the software to the client sooner, but it also helps to get results earlier. And so if a problem occurs earlier (such as the detection of a bug), it costs much less than if it is discovered later.

The investment is heavy. The return on investment will be on time. When we talk about automation, we think about “infrastructure as code” (for this we can use tools like Ansible or Chef),continuous delivery pipelines (here we are talking about Jenkins for example).The concept of “infrastructure as code” makes it possible to automate the tests on the infrastructure, to ensure that the infrastructure is continuously tested. It also allows to set up pre-production environments identical to production environments.

### M : Measurement

So what can we measure? Well we could measure well the response time of the system, which allows to know for example if the last change made to the system has improved performance or on the contrary has degraded it.

Therefore, on the developer side, it is very important that they provide on-board monitoring services in the provided applications.

From these measures, KPIs (Key Performance Indicators) can be established to answer important questions, such as:

What is the billing made today by the cloud service providers At what time we should get alarmed about the billing ? How many users have registered today? What are the incomes today? What are the operating costs? What is the number of tickets opened to the Call Center today?

### S : Sharing

The last component of CAMS is Sharing. Sharing has three components:

The visibility - Visibility is what allows everyone to see the progress of other parts of the organization. So that’s what you did.

The transparency - Transparency is what allows everyone to work towards a common goal. So that’s why we did what we did.

The transfer of knowledge - The transfer of knowledge aims to avoiding constraints in the organization and promote the collective intelligence.

# Configuration Management

After understanding the core values of DevOps let's jump into another vital concept of DevOps which is unavoidable part of DevOps and which is Configuration Management.

So, what exactly configuration management means?

Configuration Management is the process of maintaining systems, such as computer hardware and software, in a desired state. Configuration Management (CM) is also a method of ensuring that systems perform in a manner consistent with expectations over time . CM is often used with IT service management as defined by the IT Infrastructure Library (ITIL). CM is often implemented with the use of configuration management tools such as those incorporated into VMware vCenter etc..

## Configuration Management Tools

Configuration management tools enable changes and deployments to be faster, repeatable, scalable, predictable, and able to maintain the desired state, which brings controlled assets into an expected state. Below are some widely used tools for configuration management :

**Ansible** **Chef** **puppet** **Salt**

We'll deep dive into the CM tools in another blog .

Thanks & Happy Reading :)