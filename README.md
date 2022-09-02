
#Build and Deployment flow
---
## feature > master
**Pull Request**
    - Do nothing

**Push**
    - Do nothing

---
## master > deployment
**Pull Request**
- pipeline named `Pull Request name` triggers. `ref_name: master`
    - Do nothing

**Push**
- pipeline named `Merge pull request #` triggers. `ref_name: deployment`
    - *BuildContainerSnapshotImage*
        - Build a container image
            ``` 
            # skaffold build 
            ``` 
    - UpdateDeploymentSnapshotTemplate
        - Update `app-deployment` on `overlay/dev/kustomize.yaml`
            ```
            # kustomize edit set image ${IMG_NAME}=${IMG_NAME}:${IMG_VERSION}
            ```
---
## deployment > release
**Pull Request**
- pipeline named `Pull Request name` triggers. `ref_name: deployment`
    - Do nothing

**Push**
- pipeline named `Merge pull request #` triggers. `ref_name: release`
    - PreRelease
        - Create `New Tag` from `release` branch
            ```
            # ./gradlew -x test -Prelease.useAutomaticVersion=true release
            ```
- pipeline named `[Gradle Release Plugin] - pre tag commit: '1.0.0'` triggers. `ref_name: release`
    - Do nothing
- pipeline named `[Gradle Release Plugin] - pre tag commit: '1.0.0'` triggers. `ref_name: 1.0.0`
    - BuildContainerReleaseImage
        - Build a container image
            ```
            # skaffold build -q --tag 1.0.0
            ```
    - UpdateDeploymentSnapshotTemplate
        - Update `app-deployment` on `overlay/sit/kustomize.yaml`
            ```
            # kustomize edit set image ${IMG_NAME}=${IMG_NAME}:${IMG_VERSION}
            ```
- pipeline named `[Gradle Release Plugin] - new version commit: '1.0.1-SNAPSHOT'` triggers. `ref_name: release`
    - Do nothing
