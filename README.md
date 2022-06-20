| **Reporter**        | **Github Pages**   | **Azure Storage Static Website** | **AWS S3 Static Website**                                                                    |
|---------------------|--------------------|-------------------------------|----------------------------------------------------------------------------------------------|
| **Allure HTML**     | [GH Action Link](https://github.com/marketplace/actions/allure-html-reporter-github-pages) | [GH Action Link](https://github.com/marketplace/actions/allure-html-reporter-azure-website)            | [GH Action Link](https://github.com/marketplace/actions/allure-html-reporter-s3-website )      |
| **Any HTML Reports** | [GH Action Link](https://github.com/marketplace/actions/html-reporter-github-pages) | [GH Action Link](https://github.com/marketplace/actions/html-reporter-azure-website)            | [GH Action Link](https://github.com/marketplace/actions/html-reporter-aws-s3-website) |


Example workflow file [allure-html-report-aws-s3-website](https://github.com/PavanMudigonda/allure-html-reporter-s3-website/blob/main/.github/workflows/allure.yml))

# Allure HTML Test Results on AWS S3 Bucket with history action

## Usage

### `allure.yml` Example

Place in a `.yml` file such as this one in your `.github/workflows` folder. [Refer to the documentation on workflow YAML syntax here.](https://help.github.com/en/articles/workflow-syntax-for-github-actions)

As of v0.1, all [`aws s3 sync` flags](https://docs.aws.amazon.com/cli/latest/reference/s3/sync.html) are optional to allow for maximum customizability (that's a word, I promise) and must be provided by you via `args:`.

#### The following example includes optimal defaults for a public static website:

- `--acl public-read` makes your files publicly readable (make sure your [bucket settings are also set to public](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteAccessPermissionsReqd.html)).
- `--follow-symlinks` won't hurt and fixes some weird symbolic link problems that may come up.
- Most importantly, `--delete` **permanently deletes** files in the S3 bucket that are **not** present in the latest version of your repository/build.
- **Optional tip:** If you're uploading the root of your repository, adding `--exclude '.git/*'` prevents your `.git` folder from syncing, which would expose your source code history if your project is closed-source. (To exclude more than one pattern, you must have one `--exclude` flag per exclusion. The single quotes are also important!)

```yaml
name: test-results

on:
  push:
    branches:
    - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: allure-html-report-s3-website
        uses: PavanMudigonda/allure-html-reporter-aws-s3-website@main
        with:
          report_url: http://allure-report-bucket.s3-website-us-east-1.amazonaws.com
          allure_results: allure-results
          allure_history: allure-history
          allure_report: allure-report
          keep_reports: 2
          args: --acl public-read --follow-symlinks 
        env:
          AWS_S3_BUCKET: ${{ secrets.AWS_S3_BUCKET }}
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          AWS_REGION: 'us-east-1'   # optional: defaults to us-east-1
          SOURCE_DIR: 'allure-history'      # optional: defaults to entire repository
          # DEST_DIR: ${{ env.GITHUB_RUN_NUMBER }}
          
      - name: Post the link to the report # Optional
        if: always()
        uses: Sibz/github-status-action@v1
        with: 
            authToken: ${{secrets.GITHUB_TOKEN}}
            context: 'Test Results Link'
            state: 'success'
            sha: ${{ github.sha }}
            target_url: http://${{ secrets.AWS_S3_BUCKET }}.s3-website-${{ env.AWS_REGION }}.amazonaws.com/${{ github.run_number }}/index.html          
```


### Configuration

The following settings must be passed as environment variables as shown in the example. Sensitive information, especially `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`, should be [set as encrypted secrets](https://help.github.com/en/articles/virtual-environments-for-github-actions#creating-and-using-secrets-encrypted-variables) — otherwise, they'll be public to anyone browsing your repository's source code and CI logs.

| Key | Value | Suggested Type | Required | Default |
| ------------- | ------------- | ------------- | ------------- | ------------- |
| `report_url` | Your Report URL. | `argument` | **Yes** | none |
| `allure_report` | allure-report | `argument` | **No** | allure-report |
| `allure_results` | allure-results | `argument` | **No** | allure-results |
| `allure_history` | allure-history | `argument` | **No** | allure-history |
| `keep_reports` | 20 | `argument` | **No** | 20 |
| `AWS_ACCESS_KEY_ID` | Your AWS Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) | `secret env` | **Yes** | N/A |
| `AWS_SECRET_ACCESS_KEY` | Your AWS Secret Access Key. [More info here.](https://docs.aws.amazon.com/general/latest/gr/managing-aws-access-keys.html) | `secret env` | **Yes** | N/A |
| `AWS_S3_BUCKET` | The name of the bucket you're syncing to. For example, `jarv.is` or `my-app-releases`. | `secret env` | **Yes** | N/A |
| `AWS_REGION` | The region where you created your bucket. Set to `us-east-1` by default. [Full list of regions here.](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html#concepts-available-regions) | `env` | No | `us-east-1` |
| `AWS_S3_ENDPOINT` | The endpoint URL of the bucket you're syncing to. Can be used for [VPC scenarios](https://aws.amazon.com/blogs/aws/new-vpc-endpoint-for-amazon-s3/) or for non-AWS services using the S3 API, like [DigitalOcean Spaces](https://www.digitalocean.com/community/tools/adapting-an-existing-aws-s3-application-to-digitalocean-spaces). | `env` | No | Automatic (`s3.amazonaws.com` or AWS's region-specific equivalent) |
| `SOURCE_DIR` | The local directory (or file) you wish to sync/upload to S3. For example, `public`. Defaults to your entire repository. | `env` | No | `./` (root of cloned repository) |
| `DEST_DIR` | The directory inside of the S3 bucket you wish to sync/upload to. For example, `my_project/assets`. Defaults to the root of the bucket. | `env` | No | `/` (root of bucket) |

### AWS S3 Bucket folder structure sample:- Organized by Github Run Number. 
Note:- Always the index.html points to Test Results History Page 

<img width="792" alt="image" src="https://user-images.githubusercontent.com/29324338/174341322-4d72d7cf-9861-4ecd-afcb-d3f294f75a16.png">

### When click on a link 

<img width="1341" alt="image" src="https://user-images.githubusercontent.com/29324338/173705832-3954fcff-1902-45ef-bc61-3515b98f3b23.png">

### AWS S3 Static Website Sample:- Full Report, Errors, Screenshots, Trace, Video is fully visible !

<img width="1419" alt="image" src="https://user-images.githubusercontent.com/29324338/173703662-0f6a8c54-466e-4722-8344-b1c8635a57e1.png">
<img width="841" alt="image" src="https://user-images.githubusercontent.com/29324338/173703700-419a6697-e235-422f-af87-cfd4236eb6a6.png">

<img width="1405" alt="image" src="https://user-images.githubusercontent.com/29324338/173703723-00ec4828-2232-432a-8f22-4dfa074ffcb9.png">

<img width="1438" alt="image" src="https://user-images.githubusercontent.com/29324338/173703740-f2d3c1f3-1bbe-483c-a198-7c88b92f7b1f.png">


## License

This project is distributed under the [MIT license](LICENSE.md).
