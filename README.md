# EDA Workshop First Steps 

## Resources
  
  - Rulebooks: https://ansible.readthedocs.io/projects/rulebook/en/stable/rulebooks.html
  - Generic event sources: https://console.redhat.com/ansible/automation-hub/repo/published/ansible/eda/
  - Automation Decisions (EDA User Docs): https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.5/html/using_automation_decisions/index

## Ensure tools are installed and available 
  - VS Code 
    - Red Hat Ansible extension 
  - ansible-core 2.14+ 
  - ansible-builder 3.1.0 or greater or pull a prebuild Decision Env
  - rulebook-server, ~~EDA Controller~~, or AAP 2.5 w/ access to Automation Decisions 


## Running a rulebook activation w/ ansible-rulebook 

Once a rulebook is created, you can test it out using ansible-rulebook
If you've cloned this repo and are running ansible-rulebook from this directory
this should work to start a quick example: 

`ansible-rulebook --rulebook rulebooks/webhook.yml -i inventory --verbose`

### Testing a webhook activation w/ curl 

We can test the example webhook by running this command:

`curl -H 'Content-Type: application/json' -d "{\"name\": \"Jerry\"}" http://localhost:5005/endpoint`

I find it easier to edit a JSON file in VS Code versus trying to escape json on the command line, 
so I'll usually create a json file with the payload and use a command like this to load the body 
from a file:

`curl -H 'Content-Type: application/json' -d @event_payloads/name.json http://localhost:5005/endpoint`

The '@' is what tells curl to read the data from a file so we don't have to escape things.

## Time to get real 
Let's look at an easy way to print an event out so we can understand how EDA will navigate through it.  Run the `print_event.yml` rulebook.  

`ansible-rulebook --rulebook rulebooks/print_event.yml -i inventory --verbose`

Simulate an event and take a look at the payload.  Try the other payloads in the 
event_payloads directory.

### Adding some webhook security by modifying your rulebook 

Now, if you want to add some security to authorize the webhook, we have a few choices,
we will use token here, but HMAC secrets are an argument as well.

Start up the rulebook server and use the authenticated_webhook.yml rulebook.  

Test with curl and provide the token you set in the rulebook: 

`curl -H 'Content-Type: application/json' -H 'Authorization: Bearer pizzaislife' -d @event_payloads/name.json http://localhost:5005/endpoint`

## Todo
- [] Add building a custom Decision Environment to include k8s source 
- [] Build the Exercise steps for eventing with k8s source (what should that even be?  I like the Create Project -> have AAP apply resourceLimits, but is that realistic?)








```yaml
---
- name: Listen for events on a webhook
  hosts: all
  ## Define our source for events
  sources:
    - ansible.eda.webhook:
        host: 0.0.0.0
        port: 5000
  ## Define the conditions we are looking for
  rules:
    - name: Say Hello
      condition: event.payload.message == "Ansible is super cool"
  ## Define the action we should take should the condition be met
      action:
        run_playbook:
          name: say-what.yml
```

