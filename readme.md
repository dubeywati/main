
   Master branch 
# name: main
   
# on:
#   pull_request:
#     branches:
#         - 'master'
#         - 'staging'
#         - 'production'
#         - 'feature/automation'
  
# jobs: 

#   wait-for-previous:
#     runs-on: ubuntu-latest
#     steps:
#       - name: Check Previous PR Workflow Status
#         id: check-status
#         run: |
#           while true; do
#             api_url="https://api.github.com/repos/${{ github.repository }}/actions/runs?event=pull_request&status=in_progress&per_page=1"
#             api_response=$(curl -s -H "Authorization: token ${{ secrets.PAT_TOKEN }}" "$api_url")
#             total_count=$(echo "$api_response" | jq -r '.total_count')
#             echo "API url: $api_url"
#             echo "Total Count of Previous PR Workflows: $total_count"

#             if [ "$total_count" -eq 0 ]; then
#               echo "No previous PR workflows found in progress or queued."
#               exit 0
#             elif [ "$total_count" -eq 1 ]; then
#               head_branch=$(echo "$api_response" | jq -r '.workflow_runs[0].head_branch')
#               current_branch=$(echo "${{ github.head_ref }}" | sed 's/refs\/heads\///')
#               echo "Head Branch of Previous PR Workflow: $head_branch"
#               echo "Current Branch of This Workflow: $current_branch"

#               if [ "$head_branch" == "$current_branch" ]; then
#                 echo "Head branch matches the current branch. Exiting..."
#                 exit 0
#               fi
#             fi

#             echo "Waiting for the previous PR workflows to complete..."
#             sleep 20
#           done
#         env:
#           GITHUB_TOKEN: ${{ secrets.PAT_TOKEN }}

 
#   build: 
    
#     permissions:
#       contents: 'write'
#       id-token: 'write'
#       pull-requests: 'write'

#     runs-on: ubuntu-latest
#     needs: [wait-for-previous]
#     steps:
    
#     - uses: mdecoleman/pr-branch-name@1.2.0
#       id: vars
#       with:
#         repo-token: ${{ secrets.PAT_TOKEN }}
#     - run: echo ${{ steps.vars.outputs.branch }}
    
#     - name: Checkout  
#       uses: actions/checkout@v1
#       with:
#         ref: ${{ steps.vars.outputs.branch }}

#     - name: Setup .NET Core
#       uses: actions/setup-dotnet@v1
#       with:
#         dotnet-version: 6.0.x
   
#     - id: 'auth'
#       name: Authenticate to Google Cloud
#       uses: google-github-actions/auth@v1
#       with:
#         token_format: 'access_token'
#         workload_identity_provider: 'projects/454661847059/locations/global/workloadIdentityPools/github-wif-pool/providers/githubwif'
#         service_account: 'githubaction-gcr@wati-gke.iam.gserviceaccount.com'
   

#     - name: Login to GCR
#       uses: docker/login-action@v2
#       with:
#         registry: asia.gcr.io
#         username: oauth2accesstoken
#         password: ${{ steps.auth.outputs.access_token }}  

#     - name: Build server
#       run: |-
#         docker build \
#           --tag "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev106058" \
#           ./netcore-mvc

#     # - name: Login to GCR
#     #   uses: docker/login-action@v2
#     #   with:
#     #     registry: asia.gcr.io
#     #     username: oauth2accesstoken
#     #     password: ${{ steps.auth.outputs.access_token }}

#     # Push the Docker image to Google Container Registry
#     - name: Publish server non-triAL
#       run: |-
#         docker push "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev106058"

#     - name: Publish server for trial env
#       run: |-
#         docker tag "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev106058" "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev300045"
#         docker push "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev300045"
    
#     - name: Build frontend 
#       run: |-
#         docker build \
#           --tag "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev106058" \
#           ./chat

#     # Push the Docker image to Google Container Registry
#     - name: Publish frontend for live env
#       run: |-
#         docker push "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev106058"

#     - name: Publish frontend for trial env
#       run: |-
#         docker tag "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev106058" "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev300045"
#         docker push "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev300045"


#     - name: Deploy to non-trial env
#       run: |-
#         curl -X GET -H "Authorization: Bearer faIYN5f8kkjE20P7x7Vi" http://34.126.125.92:8808/api/restart_wati?name=106058

#     - name: Deploy to trial env
#       run: |-
#         curl -X GET -H "Authorization: Bearer faIYN5f8kkjE20P7x7Vi" http://34.126.125.92:8808/api/restart_wati?name=300045
 
#     - name: Executing Regression Tests for Non-Trail ENV
#       uses: convictional/trigger-workflow-and-wait@v1.6.5
#       with:
#         owner: ClareAI
#         repo: wati_test_automation
#         github_token: ${{ secrets.PAT_TOKEN }}
#         github_user: dubeywati
#         workflow_file_name: TestBuildAndRun.yml
#         ref:  new-test-automation
#         wait_interval: 10
#         client_payload: '{"branch_name":"${{ steps.vars.outputs.branch }}","PR_ISSUE_NUMBER": "${{ github.event.pull_request.number }}", "PR_REPO_OWNER": "${{ github.repository_owner }}", "PR_REPO_NAME": "${{  github.event.repository.name }}"}'
#         propagate_failure: true
#         trigger_workflow: true
#         wait_workflow: true
#         comment_downstream_url: ${{ github.event.pull_request.comments_url }}
#     - name: Executing Regression Tests for Trail ENV
#       uses: convictional/trigger-workflow-and-wait@v1.6.5
#       with:
#         owner: ClareAI
#         repo: wati_test_automation
#         github_token: ${{ secrets.PAT_TOKEN }}
#         github_user: dubeywati 
#         workflow_file_name: trail.yml
#         ref: new-test-automation
#         wait_interval: 10
#         client_payload: '{"branch_name":"${{ steps.vars.outputs.branch }}","PR_ISSUE_NUMBER": "${{ github.event.pull_request.number }}", "PR_REPO_OWNER": "${{ github.repository_owner }}", "PR_REPO_NAME": "${{  github.event.repository.name }}"}'
#         propagate_failure: true
#         trigger_workflow: true
#         wait_workflow: true
#         comment_downstream_url: ${{ github.event.pull_request.comments_url }} 



name: main
   
on:
  pull_request:
    branches:
        - 'master'
        - 'staging'
        - 'production'
        - 'feature/automation'
        - 'feature/automation-trail-testing'
  
jobs: 

  # wait-for-previous:
  #   runs-on: ubuntu-latest
  #   steps:
  #     - name: Check Previous PR Workflow Status
  #       id: check-status
  #       run: |
  #         while true; do
  #           api_url="https://api.github.com/repos/${{ github.repository }}/actions/runs?event=pull_request&status=in_progress&per_page=1"
  #           api_response=$(curl -s -H "Authorization: token ${{ secrets.PAT_TOKEN }}" "$api_url")
  #           total_count=$(echo "$api_response" | jq -r '.total_count')
  #           echo "API url: $api_url"
  #           echo "Total Count of Previous PR Workflows: $total_count"

  #           if [ "$total_count" -eq 0 ]; then
  #             echo "No previous PR workflows found in progress or queued."
  #             exit 0
  #           elif [ "$total_count" -eq 1 ]; then
  #             head_branch=$(echo "$api_response" | jq -r '.workflow_runs[0].head_branch')
  #             current_branch=$(echo "${{ github.head_ref }}" | sed 's/refs\/heads\///')
  #             echo "Head Branch of Previous PR Workflow: $head_branch"
  #             echo "Current Branch of This Workflow: $current_branch"

  #             if [ "$head_branch" == "$current_branch" ]; then
  #               echo "Head branch matches the current branch. Exiting..."
  #               exit 0
  #             fi
  #           fi

  #           echo "Waiting for the previous PR workflows to complete..."
  #           sleep 20
  #         done
  #       env:
  #         GITHUB_TOKEN: ${{ secrets.PAT_TOKEN }}

 
  build_deploy: 
    
    permissions:
      contents: 'write'
      id-token: 'write'
      pull-requests: 'write'

    runs-on: ubuntu-latest
    # needs: [wait-for-previous]
    steps:
    
    - uses: mdecoleman/pr-branch-name@1.2.0
      id: vars
      with:
        repo-token: ${{ secrets.PAT_TOKEN }}
    - run: echo ${{ steps.vars.outputs.branch }}
    
    - name: Checkout  
      uses: actions/checkout@v1
      with:
        ref: ${{ steps.vars.outputs.branch }}

    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 6.0.x
   
    - id: 'auth'
      name: Authenticate to Google Cloud
      uses: google-github-actions/auth@v1
      with:
        token_format: 'access_token'
        workload_identity_provider: 'projects/454661847059/locations/global/workloadIdentityPools/github-wif-pool/providers/githubwif'
        service_account: 'githubaction-gcr@wati-gke.iam.gserviceaccount.com'
   

    - name: Login to GCR
      uses: docker/login-action@v2
      with:
        registry: asia.gcr.io
        username: oauth2accesstoken
        password: ${{ steps.auth.outputs.access_token }}  

    - name: Build server
      run: |-
        docker build \
          --tag "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev106058" \
          ./netcore-mvc

    # - name: Login to GCR
    #   uses: docker/login-action@v2
    #   with:
    #     registry: asia.gcr.io
    #     username: oauth2accesstoken
    #     password: ${{ steps.auth.outputs.access_token }}

    # Push the Docker image to Google Container Registry
    - name: Publish server non-triAL
      run: |-
        docker push "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev106058"

    - name: Publish server for trial env
      run: |-
        docker tag "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev106058" "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev300045"
        docker push "asia.gcr.io/wati-gke/whatsapp_inbox_server:dev300045"
    
    - name: Build frontend 
      run: |-
        docker build \
          --tag "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev106058" \
          ./chat

    # Push the Docker image to Google Container Registry
    - name: Publish frontend for live env
      run: |-
        docker push "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev106058"

    - name: Publish frontend for trial env
      run: |-
        docker tag "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev106058" "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev300045"
        docker push "asia.gcr.io/wati-gke/whatsapp_inbox_chat:dev300045"


    - name: Deploy to non-trial env
      run: |-
        curl -X GET -H "Authorization: Bearer faIYN5f8kkjE20P7x7Vi" http://34.126.125.92:8808/api/restart_wati?name=106058

    - name: Deploy to trial env
      run: |-
        curl -X GET -H "Authorization: Bearer faIYN5f8kkjE20P7x7Vi" http://34.126.125.92:8808/api/restart_wati?name=300045

  Non-trail_testing:
    runs-on: ubuntu-latest
    needs: build_deploy
    steps:
    - uses: mdecoleman/pr-branch-name@1.2.0
      id: vars
      with:
         repo-token: ${{ secrets.PAT_TOKEN }}
    - run: echo ${{ steps.vars.outputs.branch }}
      
    - name: Checkout  
      uses: actions/checkout@v1
      with:
        ref: ${{ steps.vars.outputs.branch }}

    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
          dotnet-version: 6.0.x
    - name: Executing Regression Tests for Non-Trail ENV
      uses: convictional/trigger-workflow-and-wait@v1.6.5
      with:
        owner: ClareAI
        repo: wati_test_automation
        github_token: ${{ secrets.PAT_TOKEN }}
        github_user: dubeywati
        workflow_file_name: TestBuildAndRun.yml
        ref:  new-test-automation
        wait_interval: 10
        client_payload: '{"branch_name":"${{ steps.vars.outputs.branch }}","PR_ISSUE_NUMBER": "${{ github.event.pull_request.number }}", "PR_REPO_OWNER": "${{ github.repository_owner }}", "PR_REPO_NAME": "${{  github.event.repository.name }}"}'
        propagate_failure: true
        trigger_workflow: true
        wait_workflow: true
        comment_downstream_url: ${{ github.event.pull_request.comments_url }}

  Trail_testing:
    runs-on: ubuntu-latest
    needs: build_deploy
    steps:
    - uses: mdecoleman/pr-branch-name@1.2.0
      id: vars
      with:
         repo-token: ${{ secrets.PAT_TOKEN }}
    - run: echo ${{ steps.vars.outputs.branch }}
      
    - name: Checkout  
      uses: actions/checkout@v1
      with:
        ref: ${{ steps.vars.outputs.branch }}

    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
          dotnet-version: 6.0.x
    - name: Executing Regression Tests for Trail ENV
      uses: convictional/trigger-workflow-and-wait@v1.6.5
      with:
        owner: ClareAI
        repo: wati_test_automation
        github_token: ${{ secrets.PAT_TOKEN }}
        github_user: dubeywati 
        workflow_file_name: trail.yml
        ref: new-test-automation
        wait_interval: 10
        client_payload: '{"branch_name":"${{ steps.vars.outputs.branch }}","PR_ISSUE_NUMBER": "${{ github.event.pull_request.number }}", "PR_REPO_OWNER": "${{ github.repository_owner }}", "PR_REPO_NAME": "${{  github.event.repository.name }}"}'
        propagate_failure: true
        trigger_workflow: true
        wait_workflow: true
        comment_downstream_url: ${{ github.event.pull_request.comments_url }} 
>>>>>>> 642e23b (Update readme.md)
