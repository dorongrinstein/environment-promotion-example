# Control Plane - Environment Promotion Example

This example demonstrates the promotion of an application to downstream environments (development -> production) using
Control Plane and GitHub Actions. As part of the promotion process, a `Staging Workload` is deployed to test
updates before promoting to production.

The current repository contains a Node.js application with two branches, `dev` and `main`. These branches correspond to `dev` and `prod` Control Plane Orgs, respectively.

During the promotion process, the image built for development is used by the production environment and does not need to be rebuilt. This is a great time saver for lengthy build processes.

This example demonstrates the promotion between two environments - dev, main. There is no limit to the number of Orgs / environments that can be used. The example can be extended to your own unique deployment requirements.

## GitHub Actions

This project contains three GitHub Actions (in the `./.github/workflows` directory) that perform the following:

1. On a push to the `dev` branch, the application is containerized and pushed to the dev Org's private repository.
   The GVC and Workload is updated (or created if it doesn't exists) by applying the YAML contents of the files `./cpln/cpln-gvc.yaml` and `./cpln/cpln-workload.yaml`.
2. When a pull request is opened to merge the code to the `main` branch, a `staging` workload is
   updated (or created if it doesn't exists) in the same dev Org by applying the same files as step 1, except that the workload name is prefixed with `staging`. This allows the application to be reviewed and tested before being pushed to the production Org. Note that the same org is used in this case, but it could easily be factored to use one org per environement.
3. Once the pull request is accepted and the code is merged to the `main` branch, the GVC and Workload in the production Org
   is updated (or created if it doesn't exists) by applying the YAML contents of the files `./cpln/cpln-gvc-prod.yaml` and `./cpln/cpln-workload.yaml`. The main difference between the two GVC files is that the `prod` version contains the `Pull Secret` that is needed the pull the image from the dev Org. The Workload uses the image pushed earlier to the development Org.

## Control Plane Prerequisites

The following Control Plane resources are required:

- Two Control Plane Orgs representing a `dev` and `prod` environment.

## Control Plane Authentication Set Up

This example uses the Control Plane CLI to perform the necessary actions within the GitHub Actions. The CLI requires a `Service Account` with the proper permissions to perform actions against the Control Plane API.

Since actions will be performed against multiple Orgs, a `Service Account` is required in each Org.

**Perform the following steps in each Org:**

1. Follow the Control Plane documentation to create a `Service Account` and create a key. Take a note of the key. It will be used in the next section.
2. Add the Service Account to the `superusers` group. Once the GitHub Action executes as expected, a policy can be created with a limited set of permissions and the `Service Account` can be removed from the `superusers` group.

## Example Set Up

**Perform the following steps to set up the example:**

1. Fork the example into your own workspace.

2. The following variables are required and must be added as GitHub repository secrets.

```
Browse to the Secrets page by clicking `Settings` (top menu bar), then `Secrets` (left menu bar), and finally click `Actions`.
```

Add the following variables:

- `CPLN_ORG_DEVELOPMENT`: Control Plane development Org.
- `CPLN_ORG_PRODUCTION`: Control Plane production Org.
- `CPLN_GVC`: The name of the GVC.
- `CPLN_WORKLOAD`: The name of the Workload.
- `CPLN_TOKEN_DEVELOPMENT`: Development Org's Service Account Key from the previous step.
- `CPLN_TOKEN_PRODUCTION`: Production Org's Service Account Key from the previous step.
- `CPLN_IMAGE`: The name of the image that will be deployed to the development Org's private repository. The GitHub Action workflow will append the short SHA of the commit as the image tag.

3. Review the `.github/workflow/*` files. These actions will be triggered as described in the `GitHub Actions` section above.

4. Review the Control Plane YAML files that are located in the `/cpln` directory. No changes are required to execute the example.
   - The `cpln-gvc.yaml` file defines the GVC to be created/updated for the `dev` Org.
   - The `cpln-gvc-prod.yaml` file defines the Pull Secret and GVC to be created/updated for the `prod` Org.
   - The `cpln-workload.yaml` file defines the Workload to be created/updated corresponding to the dev/staging/prod Workloads.

## Running the App

After the GitHub Action has successfully deployed the application, it can be tested by following these steps:

1. Browse to the Control Plane Console (https://console.cpln.io/).
2. If necessary, select a different Org by clicking your profile circle in the upper right corner, click the Org pull-down, and select the target Org.
3. Select the GVC that was set in the `CPLN_GVC` variable.
4. Select the workload that was set in the `CPLN_WORKLOAD` variable. The `staging` Workload will be prefixed with `staging-`.
5. Click the `Open` button. The app will open in a new tab. The output of the application will be displayed.

## Notes

- The `cpln apply` command creates and updates the resources defined within the YAML file. If the name of a resource is changed, `cpln apply` will create a new resource. Any orphaned resources will need to be manually deleted.

- The Control Plane CLI commands use the `CPLN_ORG` and `CPLN_TOKEN` environment variables when needed. There is no need to add the --org or --token flags when executing CLI commands.

- The GVC definition must exists in its own YAML file. The `cpln apply` command executing the file that contains the GVC definition must be executed before any child definition YAML files (workloads, identities, etc.) are executed.

## Helper Links

GitHub

- <a href="https://docs.github.com/en/actions" target="_blank">GitHub Actions Docs</a>
