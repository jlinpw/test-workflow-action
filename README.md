# Test Workflow Action
Sample [Docker-based Github action](https://docs.github.com/en/actions/creating-actions/creating-a-docker-container-action) to automatically test a PW workflow using the [PW client](https://raw.githubusercontent.com/parallelworks/pw-cluster-automation/master/client.py). The action defines the following steps:

1. Starts the PW resources required by the workflow
2. Submits the workflow
3. Stops the PW resources started in (1)

**Note:** If the resources requested by the workflow are already running, the workflow will use them and **not** shut them down. 

The inputs to the action are defined in the `action.yml` file here and are general so they apply to almost any PW workflow. 
To add this action to a PW workflow (stored in a separate repository since this repository stores only the action), there 
are at least two steps that are done in the **workflow repository**:

1. The PW client requires the API key of the account for authentication. Store this key as a [Github secret](https://docs.github.com/en/actions/security-guides/encrypted-secrets) in the repository with your PW workflow (**not** this repository).  Navigate to your workflow on GitHub.com and then click on `Settings` > `Secrets` > `Actions` > `New Repository Secret`. This will only be possible if you are the owner of the workflow repository or have been granted permissions to edit/view repository secrets in the workflow repository, otherwise the buttons will be hidden.
3. Tell GitHub that this action needs to be triggered whenever something (that you specify) happens in the **workflow repository** by adding this action to `.github/workflows/main.yaml` in the workflow repository.  For example, the code snippet below adds this action the Github repository of a PW workflow such that the worflow is tested with every new push:

```
on: [push]

jobs:
  test-pw-workflow:
    runs-on: ubuntu-latest
    name: test-pw-workflow-beluga
    steps:
      - name: run-workflow-beluga
        id: run-beluga
        uses: parallelworks/test-workflow-action@v5
        with:
          pw-user-host: 'beluga.parallel.works'
          pw-api-key: ${{ secrets.ALVAROVIDALTO_BELUGA_API_KEY }}
          pw-user: 'alvarovidalto'
          resource-pool-names: 'gcpslurmv2'
          workflow-name: 'singlecluster_parsl_demo'
          workflow-parameters: '{"name": "PW_USER"}'
```

The last five lines of this example of `.github/workflows/main.yaml` are the 5 essential inputs to running any 
PW workflow via the PW API. These inputs will vary depending on the host (e.g. `cloud.parallel.works`), the user's
credentials (API key and username), the resource to run the workflow on, and the workflow itself (`workflow-name`
and `workflow-parameters`); as such all 5 of these lines need to be specific to a particular workflow and are 
defined in the **workflow repository** and not in the "action repository" (here).

### Notes:
1. The workflow-parameters can be downloaded from the input form in PW as shown in the screenshot below:

<div style="text-align:left;"><img src="https://drive.google.com/uc?id=11S7U2_LGAaKxxQva6tJkOhH7r8h3heiN" width="1100"></div>

2. GitHub, by default does not allow users to use their PAT to add `.github/workflows/main.yaml`.  Either add this permission
to your PAT via your account's `Settings` > `Developer Settings` > `Personal Access Tokens`, selecting your PAT, and checking 
the workflow box or use an SSH key. An easy alternative is to use the GitHub GUI to add actions by clicking on the `Actions` tab
for your repository and then clicking on the `set up a workflow yourself ->` link.
