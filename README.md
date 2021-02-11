# Blue-Green-Deployment on AWS


Blue-green deployment is a technique that reduces downtime and risk by running two identical production environments called Blue and Green.

At any time, only one of the environments is live, with the live environment serving all production traffic. For this example, Blue is currently live, and Green is idle.
As you prepare a new version of your software, deployment and the final stage of testing takes place in the environment that is not live: in this example, Green. 
Once you have deployed and fully tested the software in Green, you switch the router, so all incoming requests now go to Green instead of Blue. Green is now live, and Blue is idle.

This technique can eliminate downtime due to app deployment. In addition, blue-green deployment reduces risk: if something unexpected happens with your new version on Green, you can immediately roll back to the last version by switching back to Blue.

Blue/Green is a deployment strategy that has been around forever and can be implemented using a variety of methods, CI-CD being one of the popular ones. Other methods include Chef, Ansible.

So basically, CI-CD is a tool to implement Blue-Green Deployment
