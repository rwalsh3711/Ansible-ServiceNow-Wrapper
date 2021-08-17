## ServiceNow Change Management Wrapper for AWX/Tower

### **Info**

***Version***: v2.1

***Description***: These playbooks provide the automation for creating, validating and closing of change tasks as a wrapper to workflows in Ansible Tower/AWX.

### **File Descriptions**

| File Name | Description |
| :-------- | :---------- | 
| SN_Change_Creation.yml | This file creates a change control, collects the change number and schedules a workflow in AWX/Tower passing the change number and other required variables to the workflow. |
| SN_Change_Validate.yml | Place this playbook before automated tasks in a workflow to validate a change is approved and in implement phase prior to running those tasks. |
| SN_Change_Close.yml | Place this playbook at the end of an automated workflow to inject start time, closing time, closure code and notes in the ticket and close the change |
| MS_Teams_Failure_Notification.yml | A generic playbook that will send a notification card to MS Teams with the contents of the variable ***ERROR_MSG*** |
| change_templates/(preamble)_change.j2 | These templates define the values for the forms fields for the change. Variables can be used to populate these values. |
| change_templates/(preamble)_variables.j2 | These templates are used to pass any required variables to the workflow needed to complete the tasks. |

### **Variables**

| Variable | Example | Description | Required |
| :------- | :------ | :---------- | :------- |
| PREAMBLE | test_workflow | This variable is used to select which change and variable templates to use during change creation and workflow scheduling. | Yes |
| JOB_NAME | Test Workflow | This variable defines which workflow to schedule. The name must match the workflow name exactly. | Yes |
| START_DAY | 1/1/2021 | This variable will set the start date for the work. Will use todays date by default if undefined. | No |
| START_HOUR | 19:30 | This variable will set the start time for the work. Will use the default of 19:00 if undefined. | No |
| RUN_HOURS | 2 | The number of hours to set between start and end time. Will use 4 hours by default if undefined. | No |

### **How To Use**

This code is designed to be used via an API call. The API must pass the variables for __PREAMBLE__ and __JOB_NAME__ along with all variables needed for the workflow that is being run to the playbook __SN_Change_Creation.yml__. The API call must pass the variables in the following format:
```
{		
"extra_vars": {
        "PREAMBLE": "My_Workflow",
        "JOB_NAME": "My Workflow",
	"WORKFLOW_VAR1": "test_variable1",
        "WORKFLOW_VAR2": "test_variable2"
	}
}
```
The **SN_Change_Creation.yml** playbook will use the ***PREAMBLE*** variable to select which **(preamble)_change.j2** and **(preamble)_variables.j2** templates to use. With this information, it will create a change control ticket and schedule the requested workflow passing along the change control ticket number under the variable ***CHG_NUM*** and any other variables defined by the **(preamble)_variables.j2** template.

When designing the workflows that will be scheduled using this code, place the **SN_Change_Validate.yml** playbook just before the actual task work. This will validate the change is in _implement_ state. The playbook will fail the workflow if this is not the case or proceed if the change status is correct. This playbook also gathers the current time and passes this information along under the variable ***WORK_START*** to be used in closing the ticket later.

You will want to place the **SN_Change_Close.yml** playbook at the end of the workflow after the automation tasks. This playbook will gather the current time and assign it to the variable ***WORK_END*** and use this along with ***WORK_START*** as the actual start and end times for closing the change ticket. It will also populate the close status using the variable ***CLOSE_CODE*** or _successful_ as the default and populate the notes section with the contents of the ***CLOSE_NOTES*** variable, or _Closed_ as the default. The closing of the change ticket will only occur if the variable ***CHG_CLOSE*** has a value of 'Yes'.

### **Known Errors and Limitations**
1. You must use a ServiceNow Credential with API Read/Write Access.
2. Ensure that the **(preamble)_change.j2** template contains values for all fields required by your ServiceNow instance.
3. Ensure that the ServiceNow assignment group and user accounts are valid.