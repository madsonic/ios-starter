# Runs on circle CI
skip_docs
min_fastlane_version("2.116.0")

default_platform(:ios)

before_all do
  setup_circle_ci
end

after_all do
  clean_build_artifacts
end

desc "Push out a new beta build"
lane :beta do |options|
  get_certs_and_profiles(type: "appstore")
  build_beta(export_method: "app-store")

  upload_to_testflight(
    skip_submission: true,
    skip_waiting_for_build_processing: true
  )

  clean_build_artifacts
end

desc "Build beta app"
lane :build_beta do |options|
  build_app(
    scheme: "acmeproject_debug",
    workspace: "acmeproject.xcworkspace",
    clean: true,
    export_method: options[:export_method]
  )
end

desc "this is work in progress, nothing has been tested"
desc "Push out app store build"
lane :release do |options|
  get_certs_and_profiles(type: "appstore")
  build(export_method: "app-store")

    appstore(
    force: true, # to avoid user prompt for app verification
    submit_for_review: false,
    automatic_release: false,
    skip_metadata: true,
    skip_screenshots: true
  )

  clean_build_artifacts
end

desc "Build release app"
lane :build do |options|
  build_app(
    scheme: "acmeproject_release",
    workspace: "acmeproject.xcworkspace",
    clean: true,
    export_method: options[:export_method]
  )
end

desc "Run tests"
lane :test do
  run_tests(
    workspace: "acmeproject.xcworkspace",
    devices: ["iPhone 6s"],
    scheme: "acmeproject_debug"
  )
end

desc "Register devices for provisioning profile"
lane :register do
  # note that this action does not 'sync' the devices but
  # only adds any unregistered devices
  # removing has to be done to apple dev portal
  register_devices(
    devices_file: "./fastlane/devices.txt"
  )
end

private_lane :get_certs_and_profiles do |options|
  # development profile is sync due to the multiple signing phase of gym
  # build signing phase uses development as set out in xcproj file
  # export signing phase uses adhoc
  # only 1 value can be set in PROVISIONING_PROFILE_SPECIFIER of xcargs
  # therefore both profiles are sync as a workaround for all these xcode nonsense

  match(type: options[:type], readonly: true)
end

lane :update_build_number do
  increment_build_number
  commit_version_bump(
  message: "Version Bump",# create a commit with a custom message
  xcodeproj: "acmeproject.xcodeproj", # optional, if you have multiple Xcode project files, you must specify your main project here
  )
  version_number = get_version_number(xcodeproj: "acmeproject.xcodeproj", target: "acmeproject")
  build_number = get_build_number(xcodeproj: "acmeproject.xcodeproj")
  add_git_tag(
    tag: "v#{version_number}-beta+#{build_number}"
  )
  add_git_tag(
    tag: "v#{version_number}-release+#{build_number}"
  )
  push_to_git_remote
end

lane :update_version_number do
  increment_version_number(
  version_number: "2.18.10" 
  )
  increment_build_number(
  build_number: "1" 
  )
end


lane :renew_push_cert do
  # dev cert
  pem(development: true, app_identifier: "com.acmeproject.app")
  pem(development: true, app_identifier: "com.acmeproject.app.prices")
  pem(development: true, app_identifier: "com.acmeproject.app.test")
  pem(development: true, app_identifier: "com.acmeproject.app.test.prices")
  # prod cert
  pem(app_identifier: "com.acmeproject.app")
  pem(app_identifier: "com.acmeproject.app.prices")
  pem(app_identifier: "com.acmeproject.app.test")
  pem(app_identifier: "com.acmeproject.app.test.prices")
end
