# Release Workflow (Tag and Promote)

This repository uses immutable Git tags for production releases. ArgoCD resolves `targetRevision` from these tags, so the tag must contain the correct version references **inside** the committed files before it is created.

## Step-by-step release process

The following example promotes from the current version to a new version `vX.Y`.

**1. Update version references**

Edit the three files that contain `targetRevision` values:

```bash
# app-of-apps-application.yaml  (root Application)
# app-of-apps/values.yaml       (global.targetRevision + each cluster targetRevision)
# README.md                     (documentation examples)
```

Change every `targetRevision` from the old tag to the new tag `vX.Y`.

**2. Commit and push to main**

```bash
git add app-of-apps-application.yaml app-of-apps/values.yaml README.md
git commit -m "chore(release): promote manifests to vX.Y"
git push origin main
```

**3. Tag the pushed commit and push the tag**

The tag must be created **after** the version-bump commit so the tag contains the new references:

```bash
git tag -a vX.Y -m "Release vX.Y"
git push origin vX.Y
```

**4. Tag each cluster repository at the same version**

For every cluster repo referenced in `values.yaml`, create and push the same tag:

```bash
cd /path/to/cluster1
git tag -a vX.Y -m "Release vX.Y"
git push origin vX.Y
```

Repeat for each cluster repository.

**5. Re-apply the root Application**

The root `Application` object in the cluster still points to the previous tag. Update it:

```bash
oc apply -f app-of-apps-application.yaml
```

ArgoCD will detect the new `targetRevision`, pull the tagged commit, render the Helm chart (which now contains `vX.Y` in its values), and propagate the new version to all ApplicationSets and their generated Applications.

**6. Verify**

Confirm all applications moved to the new tag:

```bash
oc get applications -n openshift-gitops -o custom-columns=\
NAME:.metadata.name,\
REVISION:.spec.source.targetRevision,\
SYNC:.status.sync.status,\
HEALTH:.status.health.status
```

## Rollback

To roll back, re-apply the root Application with the previous tag:

```bash
# Edit app-of-apps-application.yaml and set targetRevision back to the old tag
oc apply -f app-of-apps-application.yaml
```

Because both the old and new tags are immutable, ArgoCD will switch to the previous known-good state without any Git history changes.

## Important: tag ordering

The commit sequence for a correct release is:

```
1. Update files with new version  -->  commit  -->  push
2. Tag the pushed commit          -->  push tag
3. Re-apply root Application
```

If you tag **before** updating the version references, the tag will contain old values and child applications will not move to the new version.
