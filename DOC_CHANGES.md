<!---
This file is reset every time a new release is done. This file describes changes that have not yet been released.

Example Doc Change:
### Headline for the required change
Description of the required change.
-->

### `chef_version` and `ohai_version`

see: https://docs.chef.io/release/12-6/release_notes.html#new-metadata-rb-settings

The metadata.rb DSL is extended to support `chef_version` and `ohai_version` to establish ranges
of chef and ohai versions that the cookbook supports.

When the running chef or ohai version does not match, then the chef-client run will abort with an
exception immediately after cookbooks have been synchronized before any cookbook contents are
parsed.

The format of the dependencies is based on rubygems (and implemented with rubygems code).  Pessimistic
version constraints, floor and ceiling constraints, and specifying multiple constraints are all valid.

Examples:

```
# matches any 12.x version, but not 11.x or 13.x
chef_version "~> 12"
```

```
# matches any 12.x, 13.x, etc version
chef_version ">= 12"
```

```
# matches any chef 12 version >= 12.5.1 or any chef 13 version
chef_version ">= 12.5.1", "< 14.0"
```

```
# matches chef 11 >= 11.18.4 or chef 12 >= 12.5.1 (i.e. depends on a backported bugfix)
chef_version ">= 11.18.12", "< 12.0"
chef_version ">= 12.5.1", "< 13.0"
```

As seen in the last example multiple constraints are OR'd.

There is currently no support in supermarket for making this metadata visible in /universe to
depsolvers, or support in Berksfile/PolicyFile for automatically pruning cookbooks that fail
to match.

### `ksh` resources

Use the ksh resource to execute scripts using the Korn shell (ksh) interpreter.
This resource may also use any of the actions and properties that are available
to the execute resource.

Example:
```ruby
ksh 'hello world' do
  code <<-EOH
    echo "Hello world!"
    echo "Current directory: " $cwd
    EOH
end
```

See https://docs.chef.io/release/12-6/resource_ksh.html for more info.

### `dsc_resource` resource

Added reboot_action attribute to dsc_resource.

If the DSC resource indicates that it requires a reboot, reboot_action can use the reboot resource to
either reboot immediately (:reboot_now) or queue a reboot (:request_reboot).  The default value of reboot_action is :nothing.

### `knife bootstrap --ssh-identity-file`

The --identity-file option to `knife bootstrap` has been deprecated in favor of `knife bootstrap --ssh-identity-file`
to better align with other ssh related options.

### `windows_package` resource

`windows_package` now supports more than just `MSI`. Most common windows installer types are supported including Inno Setup, Nullsoft, Wise and InstallShield. The new allowed `installer_type` values are: `inno`, `nsis`, `wise`, `installshield`, `custom`, and `msi`.

Also, while being able to download remote installers from a `HTTP` resource is not new, it looks as though the top of the docs page is incorrect stating that only local installers can be used as a source.

Example Nullsoft (`nsis`) package resource:
```
windows_package 'Mercurial 3.6.1 (64-bit)' do
  source 'http://mercurial.selenic.com/release/windows/Mercurial-3.6.1-x64.exe'
  checksum 'febd29578cb6736163d232708b834a2ddd119aa40abc536b2c313fc5e1b5831d'
end
```

Example Custom `windows_package` resource:
```
windows_package 'Microsoft Visual C++ 2005 Redistributable' do
  source 'https://download.microsoft.com/download/6/B/B/6BB661D6-A8AE-4819-B79F-236472F6070C/vcredist_x86.exe'
  installer_type :custom
  options '/Q'
end
```
Using a `:custom` package is one way to install a non `.msi` file that embeds an `msi` based installer.

Packages can now be removed without the need to include the package `source`. The relevent uninstall metadata will now be discovered from the registry.
```
windows_package 'Mercurial 3.6.1 (64-bit)' do
  action :remove
end
```
It is important that the package name used when not including the `source` is EXACTLY the same as the display name found in "Add/Remove programs" or the `DisplayName` property in the appropriate registry key:
* HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Uninstall
* HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Uninstall
* HKEY_LOCAL_MACHINE\Software\Wow6464Node\Microsoft\Windows\CurrentVersion\Uninstall

Note that if there are multiple versions of a package installed with the same display name, all packages will be removed unless a version is provided in the `version` attribute or can be discovered in the `source` installer file.
