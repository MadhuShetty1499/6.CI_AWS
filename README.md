# Project-6: Continuous Integration on AWS Cloud
Inspired by the previous project ([Project-5](https://github.com/MadhuShetty1499/5.CI_Jenkins_and_tools)) reliance on Jenkins, Nexus, Sonar, and Git for CI, I've led a transition to a cloud-native CI infrastructure using AWS services. Leveraging tools like CodeCommit, CodeBuild, CodeDeploy, and CodePipeline, this shift enhances efficiency and automation in managing code commits, builds, tests, and deployments. The move to AWS significantly reduces operational overhead, ensuring faster bug resolution and improved agility within an Agile SDLC, marking a pivotal evolution in our development practices.

### Scenario:
  - Agile SDLC.
  - Developers make regular code changes.
  - These commits need to be built and tested.
  - Usually build and release team will do this job or the Developer's responsibility to merge and integrate code.

### Problem:
  - In Agile SDLC, there will be frequent code changes.
  - Not so frequently the code will be tested, which will accumulate bugs and errors in the code.
  - Developers need to rework to fix these bugs and errors.
  - Manual build and release process. Inter team dependencies.

### Solution:
  - Build and test for every commit.
  - Automated process.
  - Notify for every build status.
  - Fix code if bugs or errors are found instantly rather than waiting.

### Problem with CI server:
  - CI server maintenance.
  - Operational overhead to maintain server like Jenkins, Nexus, Sonar, Git, etc..

### Solution:
  - Cloud services for CI to remove Operational overhead.

### Benefits:
  - Short mean time to repair.
  - Agile.
  - No human intervention.
  - Fault isolation.
  - No Ops.

### Services and Tools used:
  - AWS Code Commit - VCS.
  - AWS Code Artifact - Maven repo for dependencies.
  - AWS Code Build - Build service.
  - AWS Code Deploy - Artifact deployment service.
  - Sonar cloud - Code analysis.
  - Checkstyle analysis.
  - AWS Code Pipeline.
  - AWS S3.

### Flow Diagram:
![FlowDiagram](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/FlowDiagram.png)

### Flow of execution:
  1. Login to AWS.
  2. Code Commit:
    - Create repository.
    - Create IAM user with Code commit policy.
    - Install AWS CLI in Git bash and configure it.
    - Generate SSH keys locally.
    - Exchange keys with IAM user.
    - Migrate source code from Github repo to code commit repo.
  3. Code Artifact:
    - Create repo for maven dependencies.
    - Export Auth token.
    - Update settings.xml and pom.xml files.
  4. Sonar cloud:
    - Create account.
    - Generate token.
    - Create SSM parameters with Sonar details in AWS.
  5. Code Build:
    - Create build project.
    - Update code build role to access SSM parameter store.
  6. Create SNS topic.
  7. Create pipeline:
    - Code Commit.
    - Test code.
    - Build.
    - Deploy to S3 bucket.
  8. Test pipeline.

### Detailed steps:
  #### Code Commit:
  - AWS Code Commit => create repo => vprofile-code-repo(name) => create.
  - AWS IAM => user => create => vprofile-code-admin(name) => attach policies directly => create policy => select Code Commit => select all(full access) => add ARN => this account => type region => type repo (vprofile-code-repo) => next => give name => create => select newly created policy => create user.
  - Select newly created user => security credentials => create access keys => command line interface => download csv.
  - In git bash => $`aws configure` => give access key, secret key, region and format(json).
  - Generate SSH keys => $`ssh-keygen` => give name(PATH/vpro-codecommit_rsa) => enter.
  - Copy public key $`cat ~/.ssh/vpro-codecommit_rsa.pub` and paste it in IAM SSH pub keys.
  - Create ssh config file for authentication $`vim ~/.ssh/config` and type as in [SSH_config](https://github.com/MadhuShetty1499/6.CI_AWS/blob/ci-aws/aws-files/ssh_config_file)
  - Save and close.
  - Check the connection $`ssh git-codecommit.<region>.amazonaws.com` and see that it is successfully authenticated.
  - Clone the source code $`git clone https://github.com/MadhuShetty1499/6.CI_AWS.git`.
  - List the branches $`git branch -a` and checkout ci-aws branch $`git checkout ci-aws`.
  - Remove git remote origin $`git remote rm origin`.
  - Add AWS code commit remote origin $`git remote add origin <URL of code commit repo>`.
  - Check that remote origin is correctly configured $`cat .git/config`.
  - Push the source code to Code commit $`git push origin --all`.
  - Check the Code commit repository.
  ![Branch](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/ci-aws.png)

  #### Code Artifact:
  - AWS Code Artifact => create repo => vprofile-maven-repo(name) => select maven central store => give domain name => next => create.
  ![Maven repo](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/mavenrepo.png)
  
  #### Sonar cloud:
  - Login using Github => my account => security => give token name => generate token => copy and save it.
  - Create / open organization => analyze new project => create project manually => give name and key => next => previous version => create => copy project key and organization key and save it.
  - AWS Systems manager => parameter store => create parameter => Organization and its key as string => create parameter => HOST and its key as string(https://sonarcloud.io) => create parameter => Project and its key as string => create parameter => LOGIN and its key as secure string(sonar token) (These parameter names can be found in Sonar build spec file ----PATH---).
  ![Parameter](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/parameters.png)

  #### Code Build:
  - Open the cloned repo in VS code.
  - Open maven central repo => view connection instructions => select linux => maven => copy url of maven central repo => paste it in pom.xml (repository section at last) and also in settings.xml (profiles and mirrors).
  - Rename [sonar-buildspec.yml](https://github.com/MadhuShetty1499/6.CI_AWS/blob/ci-aws/aws-files/sonar_buildspec.yml) file to buildspec.yml and move to the root directory where pom.xml is present.
  - In buildspec.yml update export auth token command from maven central repo view connection instructions.
  - Commit and push to Code commit => check the changes made.
  - Code build => create => vpro-code-analysis(name) => select Code commit repo => ci-aws branch => Ubuntu, standard, 5.0 => edit service role name to your convenient to access it in future => use buildspec file => cloud watch => vprofie-nvir-codebuild(group name) => sonarCodeAnalysis(stream name) => build project.
  - The build will fail because, it does not have access to parameter store. So, we have to edit service role and give access.
  - IAM => create policy => systems manager => in list (describe parameters) => in read (describe document parameter, get parameter, get parameters, get parameters by path and get parameter history) => next => vprofile-parameter-read-permission(name) => create => roles => select the code build service role which was created in previous build => attach policy to code build role and also attach Code artifact read only access => Run the build job.
  ![Test](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/Test.png)

  - Check Sonar cloud.
  - In [build_buildspec.yml](https://github.com/MadhuShetty1499/6.CI_AWS/blob/ci-aws/aws-files/build_buildspec.yml) file, update export auth token command from maven central repo.
  - Code Build => create => vprofile-build-artifact(name) => select Code commit => ci-aws branch => Ubuntu, standard, 5.0 => edit service role name to your convenient to access it in future => use buildspec file => [build_buildspec.yml](https://github.com/MadhuShetty1499/6.CI_AWS/blob/ci-aws/aws-files/build_buildspec.yml) => logs => group name same as previous(vprofie-nvir-codebuild) => buildArtifact (stream name).
  - IAM => select service role which was created from previous build => add => code artifact read only access => attach.
  - Run the job.
  ![Artifact](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/Artifact.png)

  #### Code Pipeline:
  - S3 => create bucket => give unique name => same region => create => open the bucket => create folder (pipeline-artifacts).
  ![S3](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/s3.png)

  - SNS => create => standard => vprofile-pipeline-notifications(name) => create => create subscription => email => mail id => create => open your email inbox and confirm.
  - Code pipeline => create => edit service role name to your convenience to access it in future => next => source code commit => branch ci-aws => cloudwatch events => code build => build artifact => skip deploy => create pipeline => stop execution => stop.
  - Edit the pipeline => add stage after code commit => add action sonarCodeAnalysis => Code build => source Artifact => vpro-code-analysis => done.
  - Add stage at end => deploy => add action => deploy to S3 => build artifact => bucket name => key (folder name) => extract before deploy => done => save.
  - Pipeline settings => notification rule => create => vprofile-ci-notification => select all => select SNS topic => submit.
  - Release change.
  ![Pipeline](https://github.com/MadhuShetty1499/6.CI_AWS/blob/main/Images/Pipeline.png)

  - Do some changes in the code => commit and push => check the pipeline which should be triggered automatically when there is a commit.
  - check the artifact in S3 bucket.

### Credits:
  - https://github.com/hkhcoder/vprofile-project
