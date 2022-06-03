# to-tko
TO workshop content repo

**Tasks:**

(I) Manually execute TO content use cases using the free trial TO instance. 

**a. Pre-work: Research on which TO instance and app to use for workshop - All**

Worskshop attendees will create and use their respective 30-day trial TO accounts.  Registration workflow could be automated like it has been done in the Pathfinder workshop.  This TO instance comes with the **beachshirts** application that will be used for the workshop.  We will plan on integrating the workshop cluster to the attendee TO instance to get k8s integration information 

**b. Module 1 – Connect to TO and Exploration - Prabodh**

**c. Module 2 – Full Stack Observability Dashboard - Ashley**

**d. Module 3 – Explore Integrations and deep dive on Kubernetes and Slack integrations - Anil**

e. Module 4 – Explore and Create Alerts - Ashley

f. Module 5 – Explore Distributed Tracing Capabilities - Prabodh

g. Module 6 – Wavefront Administration and Best Practices - Anil 

h. Module 7 – TO Pricing 

(II) Incorporate TO content in eduk8s workshop scripts / content

**Decisions:**

**1. Trial version comes with beachshirts app.  Use this app as much as possible.  Doesn't have k8s integrations though.  Other teams (TKG, TMC, and TSM) are using Acme Fitness app.  It is a Python app, we will have to check with the TO team whether we can add wavefront agent integrations to this app.  Parallely we will work on the Spring Pet Clinic app.  We have done the local deployment but need to make sure we can containerize (use kpack), deploy, and integrate with TO instance of the attendees**

2. Which TO instance will be used by workshop participants?  Every participant will create their own trial to account that will be valid for 30 days.  Same way as the TKO workshop on pathfinder.  Put in workflow to send an email to the participants
https://pathfinder.vmware.com/v3/path/tko_path/section/step1/activity/tanzu_kubernetes_operations 
Note that you will have pathfinder cloud org access for 48 hours after workshop begins

2. How do we take current workshop content and add TO content?  We could take a branch of the Tanzu-Partner-SE/TKG-workshop-module1. Ask Eknath to give access to branch - later

3. Can we sign up for another trial version after the initial 30 days are over?  Need to try/confirm with the TO Team.  Trial can be extended past 30 days may be 2 times.


