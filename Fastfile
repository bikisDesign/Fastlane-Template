default_platform(:ios)

platform :ios do


  ############################## PRE ##############################

  before_all do

    ENV["scheme"] = "REPLACENAME"

    ensure_git_status_clean
    
    increment_build_number

  end


   ######################### PUBLIC LANES ##########################
  lane :beta_release do
    fabric_lane()
  end

  lane :production_release do 
    itc_lane()
  end

  lane :qa_release do 
    qa_lane()
  end


  ######################### PRIVATE LANES #########################
  #------------------------- Signing -------------------------#
  private_lane :codeSign do 
    cert
    sigh(force: true)
  end

  #------------------------- Crashlytics -------------------------#
  private_lane :fabric_lane do

    sigh(
      provisioning_name: "ChamberDS Enterprise REPLACENAME",
      ignore_profiles_with_different_name: true,
    )

    gym(
      configuration: "Production",
      clean: true,
      include_bitcode: false,
      output_directory: "../",
      output_name: "REPLACENAME.ipa",
      codesigning_identity: "iPhone Distribution: Chamber DS, inc",
      xcargs: "ARCHIVE=YES"
    )

    crashlytics(
      ipa_path: "../REPLACENAME.ipa",
      api_token: "6db2cd7f851e01d847ef114dfadc062f28201237",
      build_secret: "732b63d68d82c7b70f3015f5c594d53634dc04d0869bbc0eccc45dad7b4e0d72"
    )

    post_to_slack_lane(
      destination: "Crashlytics",
      channel: "client-REPLACENAME",
      url: "https://hooks.slack.com/services/replaceURL",
    )

    File.delete("../../REPLACENAME.ipa")
    File.delete("../../REPLACENAME.app.dSYM.zip")
  end

  #------------------------- QA -------------------------#
  private_lane :qa_lane do 
    #add build lane here

    post_to_slack_lane(
      destination: "Bitbucket",
      channel: "REPLACENAME-app-buildv1",
      url: "https://hooks.slack.com/services/replaceURL",
    )
    
  end

#------------------------- ITC -------------------------#
  private_lane :itc_lane do 

    ensure_git_status_clean
    
    increment_build_number
    
    build_lane(
      config: "Release",
      codeSignID: "iPhone Distribution: Chamber DS, inc",
      cert_type: "appstore",
      provisioning_name: "ChamberDS AppStore REPLACENAME",
    )

    # post_to_slack_lane
  end

  ######################### BUILD LANES #########################
  private_lane :build_lane do |options|
    
    match(
      git_url: "https://bitbucket.org/teamchamberds/url.git",
      type: options[:cert_type],
      app_identifier: "com.chamberds.REPLACENAME",
    )

    sigh(
      provisioning_name: options[:provisioning_name],
      ignore_profiles_with_different_name: true,
    )

    gym(
      configuration: options[:config],
      clean: true,
      include_bitcode: false,
      output_directory: "../",
      output_name: "REPLACENAME.ipa",
      codesigning_identity: options[:codeSignID],
      xcargs: "ARCHIVE=YES"
    )

  end

 
 ######################### SLACK LANES #########################
  private_lane :post_to_slack_lane do |options|
    scheme = ENV["scheme"]
    environment = scheme.upcase
    version = get_version_number(xcodeproj: "REPLACENAME.xcodeproj")
    build       = get_build_number(xcodeproj: "REPLACENAME.xcodeproj")
    destination = options[:destination]

    slack(
      channel: options[:channel],
      default_payloads: [:git_branch, :test_result, :lane],
      slack_url: options[:url],
      message: "A new iOS build for REPLACENAME has been submitted to *#{destination}* :rocket:",
      attachment_properties: {
        fields: [
          {
          title: "Build Number",
          value: build,
        },
        {
          title: "Version",
          value: version,
        },
          title: "Crashlytics Troubleshoot",
          value: "https://tinyurl.com/y9n24yue"
        ]
      }
    )
  end



   ############################# POST ##############################
    # This lane is called, only if the executed lane was successful
    after_all do |lane|
       
      build = Actions.lane_context[SharedValues::BUILD_NUMBER]
      commit_version_bump(
        message: "Build #{build}",
        xcodeproj: "REPLACENAME.xcodeproj",
      )
       notification(
         message: "Fastlane finished '#{lane}' successfully"
      ) 

    end 

    error do |lane, exception|
      notification(
        message:"Fastlane '#{lane}' failed with exception: '#{exception}'"
      )
    end
end
