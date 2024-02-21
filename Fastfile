fastlane_version "2.102.0"
default_platform(:ios)
 
import("brbuild_ios/fastlane/Fastfile")
 
platform :ios do
 
  build_number = ENV["BUILD_NUMBER"]
  output_name = "FlutterBeer_AdHoc_#{build_number}"
  ipa_name = "./.build/#{output_name}"
  scheme = "Runner"
 
  def kill_simulators
    Action.sh("killall -9 Simulator 2>/dev/null || echo No simulators running")
  end
 
  def setup
    build_setup(certificate_names: ["Flutter_Cert.p12"],
                provisioning_profile_names: ["Flutter_Beer_Ad_Hoc.mobileprovision"],
                should_log: true)
  end
 
  def cleanup
    build_cleanup
    clear_derived_data(derived_data_path: "./dd")
  end
 
  desc "The buildAdHocCore lane builds the FlutterBeer archive in the ad-hoc configuration"
  lane :buildAdHocCore do
    gym(scheme: "#{scheme}", configuration: "Release", output_name: "#{output_name}", clean: true, export_method: "ad-hoc",
        output_directory: "./.build", archive_path: ipa_name, derived_data_path: "./dd")
  end
 
  desc "The uploadToAppCenter lane uploads a pre-built IPA to AppCenter"
  lane :uploadToAppCenter do
      appcenter_upload(
      api_token: "APITOKENHERE",
      owner_name: "Phtiv08",
      owner_type: "user",
      app_name: "Flutter-Beer-iOS",
      file: ".build/#{output_name}.ipa",
      destinations: 'Flutter-Beer-iOS-Distribution',
      destination_type: 'group',
      notify_testers: true
    )
  end
 
  desc "The buildAdHoc lane builds the FlutterBeer archive in the ad-hoc configuration"
  lane :buildAdHoc do
    begin
      setup
      buildAdHocCore
      uploadToAppCenter
    rescue => exception
      cleanup
      raise exception
    else
      cleanup
    end
  end
end