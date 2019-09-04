## WhiteSource account/trial required. 
    - The correct login link and creds will be provided by WhiteSource on account creation
    - Addition users or Admins can be added by initial admin. 
    - In order to create and use the scanner in a pipeline we will need a “Product” and “Project” in WhiteSource to associate the scan with. 

## WhiteSource Side Setup

WhiteSource has a couple of concepts that should be considered and must be implemented prior to configuring pipelines. A “product” acts as a container for a specific product line and within a product multiple “projects” can exist. 

To add a new product to WhiteSource find the “Products” link in the main navigation panel at the top of the page in the WhiteSource UI. Under the menu select the “+ New Product” option. 
	  
![new product button](images/ws-setup.new-product-nav.png)      
 
Enter a name for the new product that you would like to create on the next page and click create. This will create the new product under your account and redirect you to the landing page for the new product. 

![new product creation page](images/ws-setup.product-creation-page.png)

We will need project(s) for tracking attribution of which area within a product a specific code scan belongs to. The Project is required. You may only have one project for a product, but it is more likely that several projects will be required to make up the overall product that is being inventoried. You may loosely think of this as a project for code repo correlation. 

On the right side of the project page, just below the navigation panel are several buttons. Furthest to the left in this grouping is the “Add Project” button which is the entry point for creating a new project under the current product.  

![new project nav](images/ws-setup.add-project.png)

Creating the project for scanning is straight-forward. There are just a few options that need to be configured. Note: all of the project creation fields are required.

1.	Select “Empty Project” type from the list of project types. 
2.	Enter the name you wish to use for the project in WhiteSource. 
3.	Ener a description of the project. 

![new project creation page](images/ws-setup.create-project.png)
 
After creating the project, you will be redirected to your main dashboard in WhiteSource. From there you can navigate back into your product or project through the main navigation bar (hover over products and select your product), or through any link on the page that references the product/project by name (various links on the dashboard). 

## Shorten the Feedback Loop || Connect Azure DevOps to WhiteSource

To make full use of the scanner we can produce generate work items directly into an Azure DevOps Board. This needs to be configured in the WhiteSource UI. Navigate to the "Admin" tab on the navigation bar and select "Issue Tracker Settings" from the Integration area. 

![admin](images/ws-setup.admin-page.png)

From the Issue Tracker Settings page the Work Items URL and Creds must be provided.  
*NOTE* To add issues to an Azure DevOps Board you will need a link similar to older VSTS links. This can be accomplished by appending your Azure DevOps organization name to the ".visualstudio.com" domain. 

> https://_orgname_.visualstudio.com

![work items](images/ws-setup.work-item-config.png)

The final piece of adding work items is to add a policy with the Match set to "By Security Vulnerability" with an appropirate level for your intented failure level and the action set to "Issue". For "Tracker Type" select "Work Items". You should see a list of the available projects to record the issues in based on the credentials provided during the Issue Tracker Settings configuration. You will also need to configure the Issue Type and Assignee on the board. Make sure to save the policy after you are redirected back to the Policy page!

![add issues](images/ws-setup.add-issues-policy.png)

