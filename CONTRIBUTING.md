# Contributing

Contributions to this repository are restricted to D2iQ personnel.

See the [Kubeaddons Contributing Documentation](https://github.com/mesosphere/kubeaddons/blob/master/CONTRIBUTING.md) which provides the baseline information about how to contribute to this repository.

Additionally see the sections below for notes about other rules and considerations for contributions.

## Addon Delete Upgrade Strategy

It's possible to add an annotation to an Addon which will trigger a "delete style upgrade" (removal and reinstallation) for the addon under certain version conditions, e.g.:

```yaml
apiVersion: kubeaddons.mesosphere.io/v1beta2
kind: ClusterAddon
metadata:
  name: dashboard
    # versions of the dashboard older than v2 are not directly compatible and so a delete uprade is needed in this case to avoid conflicts with the older resources.
    helm.kubeaddons.mesosphere.io/upgrade-strategy: "[{\"upgradeFrom\": \"<=2.0.0\", \"strategy\": \"delete\"}]"
    # to maintain compatibility with older versions of the kubeaddons controller, please duplicate this with the old key:
    helm2.kubeaddons.mesosphere.io/upgrade-strategy: "[{\"upgradeFrom\": \"<=2.0.0\", \"strategy\": \"delete\"}]"
```

As seen in the above example, if this upgrade strategy is required for any reason **it must be documented on the addon itself the reasoning/context as to why the delete strategy is used**.

## Backporting

Unless no longer supporting cert-manager v1alpha1 resources, all addon PRs that are opened against `master` must also have a backport PR opened against `release/2`. The addon chart or kudo operator must be written in a way that will support both cert-manager's v1alpha1 resource and also v1 resources.

## Testing

There are three types of tests associated with this repository.
* [ksphere-testing-framework](https://github.com/mesosphere/ksphere-testing-framework): An integration test built around a golang framework.
* [kubeaddons-tests](https://github.com/mesosphere/kubeaddons-tests): Regression tests built using kuttl.
* [E2E tests](https://github.com/mesosphere/kommander/tree/master/system-tests#system-tests): E2E tests that spin up AWS clusters and deploy projects with addons.

## Deprecation

Sometimes you may want to **deprecate an older version (or revision) of an Addon**, for instance perhaps you have several revisions of the `v1.x` release of your addon, but now `v2.x` is out and you no longer wish to maintain the older major version.

[Releases](/README.md#Releases) for this repository are responsible for snapshotting collections of revisions and will historically keep your deprecated and unsupported versions, so when you are ready to deprecate any major/minor version of your addon follow these steps:

1. Create a PR that removes all files that are no longer supported
2. Once the PR merges, a new minor release of this repository should be created indicating removal of support for the older versions

Once this is complete end users who are using older releases of `kubernetes-base-addons` will be unaffected.

## Addon Revisions

For Addons, "revisions" are a reference to the application version with an added revision count which indicates the latest iteration on that version. This enables multiple different versions of an Addon which ultimately utilize the same underlying application version so that configuration and other aspects of the Addon can change without overriding a previously released Addon.

A single file for the addon exists at `addons/<addon-name>/<addon-name>.yaml` and the revision for that addon must be adjusted forward when any changes are made.
