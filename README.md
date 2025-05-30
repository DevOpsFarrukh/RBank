<!-- # RBank
Testing workflow


<!-- name: CICD orchestrator

on:
  workflow_dispatch:

jobs:
  dev:
    name: Baking Build for Dev
    environment:
      name: dev
    runs-on: ubuntu-latest
    steps:
      - name: Run Dev Logic
        run: echo "Running Dev Logic"

      - name: Write Success Status
        run: echo "success" > dev_status.txt

      - name: Upload Dev Status
        uses: actions/upload-artifact@v4
        with:
          name: dev-status
          path: dev_status.txt

  corp4:
    name: Baking Build for Corp4
    environment:
      name: corp4
    runs-on: ubuntu-latest
    steps:
      - name: Try to Download Dev Status
        uses: actions/download-artifact@v4
        with:
          name: dev-status
        continue-on-error: true

      - name: Check Dev Result
        id: check_dev
        run: |
          if [[ -f dev_status.txt ]] && [[ "$(cat dev_status.txt)" == "success" ]]; then
            echo "Dev was successful."
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "Dev result not available or failed. Running logic."
            echo "skip=false" >> $GITHUB_OUTPUT
          fi

      - name: Exit Early Because Dev Was Successful
        if: steps.check_dev.outputs.skip == 'true'
        run: |
          echo "Dev succeeded. Skipping Corp4 logic."
          exit 0

      - name: Run Actual Corp4 Logic
        if: steps.check_dev.outputs.skip == 'false'
        run: |
          echo "Running Corp4 logic because Dev wasn't successful or hasn't run yet."
          # Add your actual logic here -->

 #  ##############################
<!-- name: CICD orchestrator

on:
  workflow_dispatch:

jobs:
  dev:
    name: Baking Build for Dev
    environment:
      name: dev
    runs-on: ubuntu-latest
    steps:
      - name: Try to Download Corp4 Status
        uses: actions/download-artifact@v4
        with:
          name: corp4-status
        continue-on-error: true

      - name: Check Corp4 Result
        id: check_corp4
        run: |
          if [[ -f corp4_status.txt ]] && [[ "$(cat corp4_status.txt)" == "success" ]]; then
            echo "CORP4 already succeeded. Exiting DEV job early."
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "CORP4 not complete or failed. Proceeding with DEV logic."
            echo "skip=false" >> $GITHUB_OUTPUT
          fi

      - name: Exit Early Because Corp4 Was Successful
        if: steps.check_corp4.outputs.skip == 'true'
        run: |
          echo "Skipping DEV logic because CORP4 was already successful."
          exit 0

      - name: Run Dev Logic
        if: steps.check_corp4.outputs.skip == 'false'
        run: echo "Running Dev Logic"

      - name: Write Success Status
        run: echo "success" > dev_status.txt

      - name: Upload Dev Status
        uses: actions/upload-artifact@v4
        with:
          name: dev-status
          path: dev_status.txt

  corp4:
    name: Baking Build for Corp4
    environment:
      name: corp4
    runs-on: ubuntu-latest
    steps:
      - name: Try to Download Dev Status
        uses: actions/download-artifact@v4
        with:
          name: dev-status
        continue-on-error: true

      - name: Check Dev Result
        id: check_dev
        run: |
          if [[ -f dev_status.txt ]] && [[ "$(cat dev_status.txt)" == "success" ]]; then
            echo "DEV already succeeded. Exiting CORP4 job early."
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "DEV not complete or failed. Proceeding with CORP4 logic."
            echo "skip=false" >> $GITHUB_OUTPUT
          fi

      - name: Exit Early Because Dev Was Successful
        if: steps.check_dev.outputs.skip == 'true'
        run: |
          echo "Skipping CORP4 logic because DEV was already successful."
          exit 0

      - name: Run Corp4 Logic
        if: steps.check_dev.outputs.skip == 'false'
        run: echo "Running Corp4 Logic"

      - name: Write Success Status
        run: echo "success" > corp4_status.txt

      - name: Upload Corp4 Status
        uses: actions/upload-artifact@v4
        with:
          name: corp4-status
          path: corp4_status.txt --> -->

<!-- ##############################
name: CICD orchestrator

on:
  workflow_dispatch:

############################
# Skipping job with if failure logic
############################

jobs:
  dev:
    if: failure()
    name: Baking Build for Dev
    environment:
      name: dev
    runs-on: ubuntu-latest
    steps:
      - name: Try to Download Corp4 and SIT1 Status
        continue-on-error: true
        run: |
          
          echo "This is dev job
        shell: bash
        
  corp4:
    name: Baking Build for Corp4
    environment:
      name: corp4
    runs-on: ubuntu-latest
    if: always()
    needs: [dev]
    steps:
      - name: Try to Download Dev and SIT1 Status
        continue-on-error: true
        run: echo "Trying to download dev-status and sit1-status if they exist"
        shell: bash
 -->

   
  
<!--  
####################################
# Skipping job with artifact  logic

####################################

name: CICD orchestrator

on:
  workflow_dispatch:

jobs:
  dev:
    name: Baking Build for Dev
    environment:
      name: dev
    runs-on: ubuntu-latest
    steps:
      - name: Try to Download Corp4 and SIT1 Status
        continue-on-error: true
        run: |
          gh_artifact_corp4_url="https://github.com/${{ github.repository }}/actions/artifacts"
          echo "Trying to download corp4-status and sit1-status if they exist"
        shell: bash

      - uses: actions/download-artifact@v4
        with:
          name: corp4-status
        continue-on-error: true

      - uses: actions/download-artifact@v4
        with:
          name: sit1-status
        continue-on-error: true

      - name: Check if Corp4 or SIT1 Already Succeeded
        id: check_others
        run: |
          if [[ -f corp4_status.txt && "$(cat corp4_status.txt)" == "success" ]] || \
             [[ -f sit1_status.txt && "$(cat sit1_status.txt)" == "success" ]]; then
            echo "Another job (CORP4 or SIT1) already succeeded. Skipping DEV."
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "No other job succeeded yet. Proceeding with DEV."
            echo "skip=false" >> $GITHUB_OUTPUT
          fi
      - name: Exit Early Because Other Job Succeeded
        if: steps.check_others.outputs.skip == 'true'
        run: |
          echo "Skipping DEV logic."
          exit 0
      - name: Run Dev Logic
        if: steps.check_others.outputs.skip == 'false'
        run: echo "Running Dev Logic"

      - name: Write Success Status
        run: echo "success" > dev_status.txt

      - name: Upload Dev Status
        uses: actions/upload-artifact@v4
        with:
          name: dev-status
          path: dev_status.txt

  corp4:
    name: Baking Build for Corp4
    environment:
      name: corp4
    runs-on: ubuntu-latest
    steps:
      - name: Try to Download Dev and SIT1 Status
        continue-on-error: true
        run: echo "Trying to download dev-status and sit1-status if they exist"
        shell: bash

      - uses: actions/download-artifact@v4
        with:
          name: dev-status
        continue-on-error: true

      - uses: actions/download-artifact@v4
        with:
          name: sit1-status
        continue-on-error: true

      - name: Check if Dev or SIT1 Already Succeeded
        id: check_others
        run: |
          if [[ -f dev_status.txt && "$(cat dev_status.txt)" == "success" ]] || \
             [[ -f sit1_status.txt && "$(cat sit1_status.txt)" == "success" ]]; then
            echo "Another job (DEV or SIT1) already succeeded. Skipping CORP4."
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "No other job succeeded yet. Proceeding with CORP4."
            echo "skip=false" >> $GITHUB_OUTPUT
          fi
      - name: Exit Early Because Other Job Succeeded
        if: steps.check_others.outputs.skip == 'true'
        run: |
          echo "Skipping CORP4 logic."
          exit 0
      - name: Run Corp4 Logic
        if: steps.check_others.outputs.skip == 'false'
        run: echo "Running Corp4 Logic"

      - name: Write Success Status
        run: echo "success" > corp4_status.txt

      - name: Upload Corp4 Status
        uses: actions/upload-artifact@v4
        with:
          name: corp4-status
          path: corp4_status.txt

  sit1:
    name: Baking Build for SIT1
    environment:
      name: sit1
    runs-on: ubuntu-latest
    steps:
      - name: Try to Download Dev and Corp4 Status
        continue-on-error: true
        run: echo "Trying to download dev-status and corp4-status if they exist"
        shell: bash

      - uses: actions/download-artifact@v4
        with:
          name: dev-status
        continue-on-error: true

      - uses: actions/download-artifact@v4
        with:
          name: corp4-status
        continue-on-error: true

      - name: Check if Dev or Corp4 Already Succeeded
        id: check_others
        run: |
          if [[ -f dev_status.txt && "$(cat dev_status.txt)" == "success" ]] || \
             [[ -f corp4_status.txt && "$(cat corp4_status.txt)" == "success" ]]; then
            echo "Another job (DEV or CORP4) already succeeded. Skipping SIT1."
            echo "skip=true" >> $GITHUB_OUTPUT
          else
            echo "No other job succeeded yet. Proceeding with SIT1."
            echo "skip=false" >> $GITHUB_OUTPUT
          fi
      - name: Exit Early Because Other Job Succeeded
        if: steps.check_others.outputs.skip == 'true'
        run: |
          echo "Skipping SIT1 logic."
          exit 0
      - name: Run SIT1 Logic
        if: steps.check_others.outputs.skip == 'false'
        run: echo "Running SIT1 Logic"

      - name: Write Success Status
        run: echo "success" > sit1_status.txt

      - name: Upload SIT1 Status
        uses: actions/upload-artifact@v4
        with:
          name: sit1-status
          path: sit1_status.txt -->