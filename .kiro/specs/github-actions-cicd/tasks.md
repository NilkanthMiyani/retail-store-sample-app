# Implementation Plan

- [x] 1. Create workflow file structure and basic configuration




  - Create `.github/workflows/deploy.yml` file with workflow name "Deploy to ECR and Update Helm Charts"
  - Configure workflow to trigger on push to gitops branch with path filter for `src/**`
  - Add workflow_dispatch trigger for manual execution
  - Set up environment variables at workflow level for AWS_REGION and AWS_ACCOUNT_ID from secrets
  - _Requirements: 5.1, 5.2, 5.3_

- [x] 2. Implement change detection job




  - Create `detect-changes` job that runs on ubuntu-latest
  - Add checkout step with fetch-depth: 0 to access full git history
  - Implement logic to detect manual trigger (workflow_dispatch) and mark all services for build
  - Implement git diff logic using `git diff --name-only HEAD^ HEAD` to compare with previous commit
  - Add grep commands with `-q` flag to check for changes in each service directory (src/ui/, src/catalog/, src/cart/, src/checkout/, src/orders/)
  - Set output variables (build_ui, build_catalog, build_cart, build_checkout, build_orders) as "true" or "false" strings
  - Add echo statements to log which services were detected for building
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 1.7_

- [x] 3. Implement Docker build and push for UI service





  - Create `build-ui` job that depends on detect-changes job with `needs: detect-changes`
  - Add conditional execution: `if: needs.detect-changes.outputs.build_ui == 'true'`
  - Add checkout step to access source code
  - Configure AWS credentials using aws-actions/configure-aws-credentials@v4 with secrets
  - Add ECR login step using aws-actions/amazon-ecr-login@v2 and capture registry output
  - Create ECR repository retail-store-ui if it doesn't exist using AWS CLI with error suppression
  - Calculate SHORT_SHA using `echo ${{ github.sha }} | cut -c1-7`
  - Build Docker image with tag format: `{ECR_REGISTRY}/retail-store-ui:{SHORT_SHA}` using docker build command
  - Push Docker image to ECR using docker push command
  - Add echo statement to display pushed image URI
  - _Requirements: 2.1, 2.2, 2.4, 2.5, 3.1, 3.2, 3.3, 3.4, 3.5, 6.2, 6.3_

- [x] 4. Implement Docker build and push for Catalog service





  - Create `build-catalog` job with same structure as build-ui
  - Add conditional execution: `if: needs.detect-changes.outputs.build_catalog == 'true'`
  - Configure AWS credentials and ECR login
  - Create ECR repository retail-store-catalog if needed
  - Build Docker image from src/catalog/Dockerfile with SHORT_SHA tag
  - Push image and log the URI
  - _Requirements: 2.1, 2.2, 2.4, 2.5, 3.1, 3.2, 3.3, 3.4, 3.5_

- [x] 5. Implement Docker build and push for Cart service





  - Create `build-cart` job with same structure as build-ui
  - Add conditional execution: `if: needs.detect-changes.outputs.build_cart == 'true'`
  - Configure AWS credentials and ECR login
  - Create ECR repository retail-store-cart if needed
  - Build Docker image from src/cart/Dockerfile with SHORT_SHA tag
  - Push image and log the URI
  - _Requirements: 2.1, 2.2, 2.4, 2.5, 3.1, 3.2, 3.3, 3.4, 3.5_

- [x] 6. Implement Docker build and push for Checkout service




  - Create `build-checkout` job with same structure as build-ui
  - Add conditional execution: `if: needs.detect-changes.outputs.build_checkout == 'true'`
  - Configure AWS credentials and ECR login
  - Create ECR repository retail-store-checkout if needed
  - Build Docker image from src/checkout/Dockerfile with SHORT_SHA tag
  - Push image and log the URI
  - _Requirements: 2.1, 2.2, 2.4, 2.5, 3.1, 3.2, 3.3, 3.4, 3.5_

- [x] 7. Implement Docker build and push for Orders service





  - Create `build-orders` job with same structure as build-ui
  - Add conditional execution: `if: needs.detect-changes.outputs.build_orders == 'true'`
  - Configure AWS credentials and ECR login
  - Create ECR repository retail-store-orders if needed
  - Build Docker image from src/orders/Dockerfile with SHORT_SHA tag
  - Push image and log the URI
  - _Requirements: 2.1, 2.2, 2.4, 2.5, 3.1, 3.2, 3.3, 3.4, 3.5_

- [x] 8. Implement Helm chart update job





  - Create `update-helm-charts` job with needs: [detect-changes, build-ui, build-catalog, build-cart, build-checkout, build-orders]
  - Add conditional: `if: always() && (needs.build-ui.result == 'success' || needs.build-catalog.result == 'success' || needs.build-cart.result == 'success' || needs.build-checkout.result ==
  - Call update_values function for each service that was successfully built
  - Add logging to show which values.yaml files were updated
  - _Requirements: 4.1, 4.2, 4.3, 4.4, 6.4_




- [ ] 9. Implement Git commit and push logic

  - Configure Git user identity as github-actions[bot]
  - Add all modified values.yaml files to Git staging area
  - Create commit with message format: "Update Helm charts with images from commit {SHORT_SHA}"
  - Handle case where no changes exist (skip commit)




  - Push changes to gitops branch
  - Add error handling for push failures
  - _Requirements: 4.5, 4.6_

- [ ] 10. Add comprehensive error handling and logging

  - Add descriptive step names to all workflow steps
  - Ensure Docker build steps display build progress
  - Add echo statements to log ECR image URIs after push
  - Add logging for Helm chart file modifications
  - Verify error messages are captured in workflow logs for all failure scenarios
  - _Requirements: 6.1, 6.2, 6.3, 6.4, 6.5_

- [ ] 11. Create workflow documentation



  - Document required GitHub secrets in README or separate documentation
  - Document IAM permissions required for AWS credentials
  - Add usage examples for manual workflow trigger
  - Document troubleshooting steps for common failures
  - _Requirements: 5.4_