# Building Machine Images

## Packer

This solution uses [Packer](https://www.packer.io/) to create an Amazon Machine Image (AMI). This AMI forms the basis for EC2/EMR environments that investigators use for their research.

To install Packer, please see their installation [instructions](https://www.packer.io/intro/getting-started/install.html).

## Preparation

For the RStudio image to function, a few files not in the repository need to be included in the build by placing them
in the `config/infra/files/rstudio` folder (note this folder is ignored by `git` to minimize the risk of files being
committed to source control).

- `secret.txt`: A single-line text file that contains the JWT secret from the Service Workbench deployment, which can
  be found in Parameter Store at `/$stage/$solution/jwt/secret`, where `$stage` is the stage name for the
  environment and `$solution` is the solution name.

- `cert.key`: The TLS private key in PEM format for the RStudio domain.

- `cert.pem`: TLS certificate chain for the above in PEM format, with the intermediate certs first and the root last. This chain should therefore look as follows:

-----BEGIN CERTIFICATE-----<br />
_Main Certificate_<br />
-----END CERTIFICATE-----<br />
-----BEGIN CERTIFICATE-----<br />
_Intermediate Certificate_<br />
-----END CERTIFICATE-----<br />
-----BEGIN CERTIFICATE-----<br />
_Root Certificate_<br />
-----END CERTIFICATE-----<br />

The current implementation assigns hostnames to RStudio instances with the form `rstudio-$env.$domain_name` where `$env`
is the environment identifier for the workspace, and `$domain_name` is the custom domain used for Service Workbench. This means that all certificates
(the private key and the certificate chain mentioned above) must be for the wildcard (`*.$domain_name`). Failing to do so would cause nginx to not start in the EC2 instance that is backing the RStudio environment. You could also check if nginx setup is successful by running "systemctl restart nginx" on this EC2 instance.

This also means that Service Workbench must be deployed
with a custom domain for RStudio to work properly. In order to configure your custom domain name, please override and specify the following config settings in your `main/config/settings/$stage.yml` file:

1. domainName
2. certificateArn
3. hostedZoneId

## Package and Deploy

To build Amazon Machine Images:

```bash
$ pnpx sls build-image -s $STAGE
```

Note that if no configuration is provided the default VPC will be used to create AMIs, and if
no default VPC exists AMI creation will fail. To instead provide a VPC and subnet to use,
create a `config/settings/$STAGE.yml` file with ID values (see `example.yml` for an example).
