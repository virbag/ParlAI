# CI for Open Source Project

<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [CI for Open Source Project](#ci-for-open-source-project)
  - [Instructions](#instructions)
  - [Outcome](#outcome)
  - [Notes](#notes)
    - [Instruction-Specific](#instruction-specific)
    - [General](#general)

<!-- /code_chunk_output -->


## Instructions
1. Fork https://github.com/facebookresearch/ParlAI
2. Convert the existing CircleCI pipeline to BitBucket syntax.
  a. The CI pipeline/workflow will include anything that is relevant to the project you forked.
  b. For available pipeline configuration options, see the BitBucket documentation https://support.atlassian.com/bitbucket-cloud/docs/configure-bitbucket-pipelinesyml/
3. In a separate file, describe what is running as part of the CI pipeline and why you chose to include it.
  a. You can also describe any thoughts, dilemmas, challenges you had
  b. Please name the file HELM_README.txt in the base directory
  c. Describe any pipeline artifacts and their life cycles.
4. Containerize the website using Docker
  a. Please name the docker file Dockerfile.Helm
  b. Website source code is located in the `website/` directory
5. Add the github user charltonaustin to your git repository
6. Email Helm to let them know that you have finished

## Outcome
Functional pipeline that builds ParlAI and an image for the associated website.

## Notes

### Instruction-Specific
2. 
  - Removed architecture/OS specific builds because I'm not sure that's possible with BitBucket Cloud Pipelines.
  - Emplaced a parallel block for build steps in case I'm wrong about architecture/OS combinations. In some other tools I'd have done this with a matrix build configuration.
  - TODO: Match trigger pattern for image build/push step to 'commit' | or 'PR status check' on main only to avoid feature/release branches causing image publication. The approach partly depends on how the team works (following ParlAI's or a GitFlow model we _should_ do this)
3.
  a. This is my first exposure to BitBucket Pipelines. Costraints on executor architecture/OS were a surprise, as was lack of GPU accelerated options. Other red-herrings from the start were:
    - I picked a point in ParlAI's history were their build was broken. I could have used a tag to get a success but opted to stick with the main branch.
    - Setup.py does a transform of requirements.txt and that transform is broken (splits on == in package pinning lines) which yields partial/broken results when installing via setup.py. I opted to stick with "pip install -r requirements.txt" given that I've got little to do with the dev team. A possibly better alternative would be to push toward PipEnv and use Pipfile/Pipfile.lock to control dependencies. If that's not possible, fix the parsing logic in setup.py.
  b. Emplaced as HELM_README.md
  c. The pipeline produces a website.zip artifact and an OCI image which is meant to be pushed to an image registry. I didn't collect test output although some folks like to do that and publish it via http. I also didn't implement the image push but things to sort out in order to do so are:
  - Secrets handling (credentials to push)
  - Which registry makes sense
4.
   a) Emplaced Dockerfile.Helm
   b) Resulting image that can be run with docker run -p 8088:8000 website 

### General

The upstream 'website' command uses s3cmd and passes S3_ACCESS_KEY/S3_SECRET_KEY via env -- far better to have a builder/deployer role that has IAM perms for the operations necessary vice using these.

Tests are only partially effective in this pipeline since I didn't reproduce the architecure/OS and accelerated environments.

I didn't include any 'deployment' stages as I'd prefer to stop at 'delivery'. The two are separate but related concerns.
