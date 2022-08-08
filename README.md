# tap-install

- kapp deploy -a tap-manager -f ./app -n tap-install 
- kapp deploy -a tap-manager-post-install -f ./post-app -n tap-install 

## tap-values.yml
Notes:
- exclude tekton-source-pipelinerun template, since this is customized in the post-packaging sub-folder

```
ootb_templates:
  excluded_templates:
    - 'tekton-source-pipelinerun'
```
