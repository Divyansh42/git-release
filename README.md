# Git Release

It is typical to create a Git tag at the moment of release to introduce a checkpoint in your source code history,
but in most cases users will need compiled objects or other assets output, not just the raw source code.

Git Releases are a way to track deliverables in your project. Consider them a snapshot in time of the source,
build output, artifacts, and other metadata associated with a released version of your code.

This `task` can be used to make a git release to the git supported platforms.
Currently this task only have the support for :
- `Github`
- `Gitlab`

Task can also be used to upload `assets` including `binaries` of the released version, with the release.

## Install the Task

```
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/v1beta1/git-actions/git-release/git-release.yaml
```

## Parameters

- **PLATFORM**: The `Git` platform where release have to be made. 
    Supported platforms are `Github` and `Gitlab` (_default:_`Github`).
- **TAG_NAME**: The tag where the release will be created from (_e.g:_`v1.0.0`).
- **NAME**: The Name of the release (_e.g:_`First release`).
- **DESCRIPTION**: A short description of the release (_default:_`""`).
- **OWNER**: The `user name` of the Github owner (_required only in case of `Github`_).
- **REPO_NAME**: Full name of the `repository` where the release is to made (_required only in case of `Github`_).
- **BRANCH**: Branch from the repository on which release has to be made (_required only in case of `Github`_).
- **RELEASE_REF**: It can be a commit SHA, another tag name, or a branch name (_required only in case of `Gitlab`_).
- **PROJECT_ID**: The id of the project (_required only in case of `Gitlab`_).
- **UPLOAD_ASSET_NAME**: The name of the asset/link that needs to be uploaded (_default:_`""`).
- **UPLOAD_ASSET_URL**: The uplaod URL for the hosted asset (_required only in case of `Gitlab`_).
- **git-token-secret**: The name of the `secret` holding the git-token (_default:_`git-token`).
- **git-token-secret-key**: The name of the `secret key` holding the git-token (_default:_`api-token`).


## Workspace

- **upload**: To mount file which has to be uploaded with the release, required only in case of `Github`.


## Secrets
* `Secret` to provide the `credentials` of the Git profile. `Username` and `Password` are the required fields.
* `Secret` to provide API `access token` of the Github/Gitlab.

Check [this](https://docs.gitlab.com/ee/user/profile/personal_access_tokens.html) to get API access token for `Gitlab`.

Check [this](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line) to get API access token for `Github`.


## Usage

This task will use the `Service Account` with access to the secrets containing git platform `credentials`, this will authorize it to respective git platform. 

If asset is to be uploaded with the release on the `github`, then any volume (supported in workspace) can be mounted in the workapce containing 
the file that need to be uploaded.


At present, `Gitlab` doesn't provide the functionality to upload any file to the release directly, however file that is hosted on any platform (i.e `aws s3` or `gitlab`) can be uploaded with the release by providing the hosted file `URL path` as the param to the task.

To make a release put all the required params in the Taskrun, add required secrets and release will be done.

Secrets can be created as follows:
```
apiVersion: v1
kind: Secret
metadata:
  name: git-secret
type: Opaque
stringData:
  api-token: $(api_access_token)
```

```
apiVersion: v1
kind: Secret
metadata:
  name: git-creds
  annotations:
    tekton.dev/docker-0: https://gitlab.com # Should be changed to github.com when required
type: kubernetes.io/basic-auth
stringData:
  username: $(username)
  password: $(password)
```

This [example](../git-actions/git-release/example/taskrun) can be referred to create Taskrun for Github and Gitlab.

This example uses configMap for mounting upload file to the workspace, configMap can be created as follows:

```
kubectl create configmap upload-file --from-file=sample.zip
``` 

[This](../git-actions/git-release/example/service-account.yaml) can be referred to create `service account`.


### Note 


- If case asset is not to be uploaded with the release, then `emptyDir` needs to be mounted in the `workspace` as shown below:

    ```
    workspaces:
      - name: upload-file
        emptyDir: {}
    ```