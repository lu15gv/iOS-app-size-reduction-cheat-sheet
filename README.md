# iOS app size-reduction cheat sheet
Here you will find some advices and configurations to reduce iOS app size.

# General recommendations
- Use `final class` attribute in classes if no other class can inherit from it.
- Use `private` whenever you can.
- Avoid `open`/`public` if you don’t need them.
- Use static libraries instead of dynamic ones. 
    -  This will only reduce the size if you don't have the `-ObjC` flag, if you do, don't remove it! you don't want to screw up the linking of your project, right?
    -  Independently of `-ObjC` flag, I still recommend that you migrate to static instead of dynamic. This will reduce the launch time of your app.
- Find and remove unused code. You can use [Periphery](https://github.com/peripheryapp/periphery) for that.
- Inspect your .ipa to find any duplicated or unexpected assets
    - First, make an archive an export the .ipa, if you don't know how to do it, check this question in [stackoverflow](https://stackoverflow.com/questions/5499125/how-to-create-ipa-file-using-xcode)
    - Find the .ipa and change the extension to .zip, then unzip it
    - Inspect it.
    - Remove duplicate asstes from your assets catalog, or your bundles.
    - Remove or exclude from Release files that don't contribute to the final version of the .ipa. For example README.md, JSON mockups, etc.
- If you have string files (eg Localizables), remove the comments. Thanks [uber-mahyar2](https://github.com/uber-mahyar2) for the advice.

# Build Settings
### Clang
- Select your target > Build Setting > Apple Clang - Code generator > Optimization Level > Release > -Oz 

#### Swift compiler configuration

- Select your target > Build Setting > Swift Compiler - Code generator > Optimization Level > Release > -Osize 
- Select your target > Build Setting > Swift Compiler - Code generator > Compilation Moode > Release > Whole Module

### Assets

- Select your target > Build Setting > Asset Catalog Compiler - Options > Optimization > Release > space 

# CocoaPods
We should apply the above settings to all of our CocoaPods targets. Doing it manually will take forever, and they will be overwritten with every `pod install`

- Add the following `post_install` at the end of your podfile to automatically apply the settings along with the `pod install`

```ruby
post_install do |installer|
  installer.pods_project.targets.each do |target|
    target.build_configurations.each do |config|
      if config.name == 'Release'
        config.build_settings['SWIFT_OPTIMIZATION_LEVEL'] = '-Osize'
        config.build_settings['ASSETCATALOG_COMPILER_OPTIMIZATION'] = 'space'
        config.build_settings['GCC_OPTIMIZATION_LEVEL'] = 'z'
        config.build_settings['SWIFT_COMPILATION_MODE'] = 'wholemodule'
      end
    end
  end
end
```
- If you want to move to static libraries use `use_frameworks! :linkage => :static` instead of `use_frameworks`

```ruby
target 'YourTargertName' do
  use_frameworks! :linkage => :static
  # your pods
end
```

- If you don't have at least CocoaPods 1.9.0, you will have to use `use_modular_headers!`
```ruby
target 'YourTargertName' do
  use_modular_headers!
  # your pods
end
```
