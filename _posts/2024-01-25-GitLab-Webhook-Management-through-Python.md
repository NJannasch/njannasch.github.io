---
categories:
  - Software Development
  - Coding
tags:
  - python
  - requests
  - REST API
  - GitLab
---
## GitLab Webhook Management through Python
When you have a lot of GitLab projects, changing webhook settings one by one can be really slow and boring, especially if you have more than a thousand projects. To make this easier, I made a Python script. This script helps you quickly remove webhooks that have a certain keyword in them, and it does it all by itself.

I derived this script functionality from GitLab's website behavior.

```python
""" Small helper script to delete webhooks with specific keyword """
import requests

# Replace 'your_access_token' with your actual GitLab access token
GITLAB_TOKEN = '<adjust>'
GITLAB_API_URL = 'https://gitlab.com/api/v4/'

KEYWORD = 'YourKeyword'  # Keyword to be searched for in Webhook URL

# Headers for authentication
headers = {
    'Private-Token': GITLAB_TOKEN
}

def get_all_projects():
    """ Get all membership projects """
    projects = []
    page = 1
    while True:
        # Only query repos where I am member of
        response = requests.get(f"{GITLAB_API_URL}projects?page={page}&per_page=100&membership=true", headers=headers)
        if response.status_code != 200:
            print("Failed to fetch projects")
            break
        data = response.json()
        if not data:
            break
        projects.extend(data)
        page += 1
    return projects

def get_project_webhooks(project_id):
    """ Get the available webhooks for certain project with project_id """
    response = requests.get(f"{GITLAB_API_URL}projects/{project_id}/hooks", headers=headers)
    if response.status_code != 200:
        print(f"Failed to fetch webhooks for project {project_id}")
        return []
    return response.json()

def delete_project_webhook(project_id, hook_id):
    """ Delete a certain webhook from a certain project """
    response = requests.delete(f"{GITLAB_API_URL}projects/{project_id}/hooks/{hook_id}", headers=headers)
    if response.status_code != 204:
        print(f"Failed to delete webhooks for project {project_id}, {hook_id}")
    return []

def main():
    projects = get_all_projects()
    # Iterate over each project and delete webhook which contains keyword
    for project in projects:
        project_id = project['id']
        project_name = project['name']
        webhooks = get_project_webhooks(project_id)
        print(f"Project: {project_name} (ID: {project_id})")
        for webhook in webhooks:
            print(f"  Webhook: {webhook['url']}")
            if KEYWORD in webhook['url']:
                delete_project_webhook(project_id, webhook['id'])


if __name__ == "__main__":
    main()
```

### Conclusion
Leveraging Python for automation presents a significant advantage in mitigating the time-intensive nature of such tasks. This script not only serves as a practical tool for webhook deletion but also as a foundational element for further automation initiatives within GitLab environments.
