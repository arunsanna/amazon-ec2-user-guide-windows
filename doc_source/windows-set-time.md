# Setting the Time for a Windows Instance<a name="windows-set-time"></a>

A consistent and accurate time reference is crucial for many server tasks and processes\. Most system logs include a time stamp that you can use to determine when problems occur and in what order the events take place\. If you use the AWS CLI or an AWS SDK to make requests from your instance, these tools sign requests on your behalf\. If your instance's date and time are not set correctly, the date in the signature may not match the date of the request, and AWS rejects the request\. We recommend that you use Coordinated Universal Time \(UTC\) for your Windows instances\. However, you can use a different time zone if you want\.


+ [Changing the Time Zone](#windows-changing-time-zone)
+ [Configuring Network Time Protocol \(NTP\)](#windows-configuring-ntp)
+ [Configuring Time Settings for Windows Server 2008 and later](#windows-persisting-time-changes-w2k8)
+ [Related Topics](#server-time-related-topics)

## Changing the Time Zone<a name="windows-changing-time-zone"></a>

Windows instances are set to the UTC time zone by default\. you can change the time to correspond to your local time zone or a time zone for another part of your network\.

**To change the time zone on an instance**

1. From your instance, open a Command Prompt window\.

1. Identify the time zone to use on the instance\. To get a list of time zones, use the following command: tzutil /l\. This command returns a list of all available time zones, using the following format:

   ```
   display name
   time zone ID
   ```

1. Locate the time zone ID to assign to the instance\.

1. Assign the time zone to the instance by using the following command:

   ```
   tzutil /s "Pacific Standard Time"
   ```

   The new time zone should take effect immediately\.

## Configuring Network Time Protocol \(NTP\)<a name="windows-configuring-ntp"></a>

Windows instances use the time\.windows\.com NTP server to configure the system time\. We recommend that you configure your instance to use the Amazon Time Sync Service\. This service uses a fleet of satellite\-connected and atomic reference clocks in each AWS Region to deliver accurate current time readings of the Coordinated Universal Time \(UTC\) global standard\. The Amazon Time Sync Service automatically smooths any leap seconds that are added to UTC\. This service is available at the `169.254.169.123` IP address for any instance running in a VPC, and your instance does not require internet access to use this service\.

**To verify the NTP configuration**

1. From your instance, open a Command Prompt window\.

1. Get the current NTP configuration by typing the following command:

   ```
   w32tm /query /configuration
   ```

   This command returns the current configuration settings for the Windows instance\.

1. \(Optional\) Get the status of the current configuration by typing the following command:

   ```
   w32tm /query /status
   ```

   This command returns information such as the last time the instance synced with the NTP server and the poll interval\.

**To change the NTP server to use the Amazon Time Sync Service**

1. From the Command Prompt window, run the following command:

   ```
   w32tm /config /manualpeerlist:169.254.169.123 /syncfromflags:manual /update
   ```

1. Verify your new settings by using the following command:

   ```
   w32tm /query /configuration
   ```

   In the output that's returned, verify that `NtpServer` displays the `169.254.169.123` IP address\.

You can change the instance to use a different set of NTP servers if you need to\. For example, if you have Windows instances that do not have internet access, you can configure them to use an NTP server located within your private network\. Your instance's security group must allow outbound UDP traffic on port 123 \(NTP\)\.

**To change the NTP servers**

1. From the Command Prompt window, run the following command:

   ```
   w32tm /config /manualpeerlist:comma-delimited list of NTP servers /syncfromflags:manual /update
   ```

   Where *comma\-delimited list of NTP servers* is the list of NTP servers for the instance to use\.

1. Verify your new settings by using the following command:

   ```
   w32tm /query /configuration
   ```

## Configuring Time Settings for Windows Server 2008 and later<a name="windows-persisting-time-changes-w2k8"></a>

When you change the time on a Windows instance, you must ensure that the time persists through system restarts\. Otherwise, when the instance restarts, it reverts back to using UTC time\. For Windows Server 2008 and later, you can persist your time setting by adding a RealTimeIsUniversal registry key\.

**To set the RealTimeIsUniversal registry key**

1. From the instance, open a Command Prompt window\.

1. Use the following command to add the registry key:

   ```
   reg add "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /v RealTimeIsUniversal /d 1 /t REG_DWORD /f
   ```

1. If you are using a Windows Server 2008 AMI \(*not* Windows Server 2008 R2\) that was created before February 22, 2013, you should verify that the Microsoft hotfix [KB2800213](http://support.microsoft.com/default.aspx?scid=kb;EN-US;2800213) is installed\. If this hotfix is not installed, install it\. This hotfix resolves a known issue in which the RealTimeIsUniversal key causes the Windows CPU to run at 100% during Daylight savings events and the start of each calendar year \(January 1\)\.

   If you are using an AMI running Windows Server 2008 R2 \(*not* Windows Server 2008\), you must verify that the Microsoft hotfix [KB2922223](https://support.microsoft.com/en-us/help/2922223/you-cannot-change-system-time-if-realtimeisuniversal-registry-entry-is) is installed\. If this hotfix is not installed, install it\. This hotfix resolves a known issue in which the RealTimeIsUniversal key prevents the system from updating the CMOS clock\.

1. \(Optional\) Verify that the instance saved the key successfully using the following command:

   ```
   reg query "HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation" /s
   ```

   This command returns the subkeys for the TimeZoneInformation registry key\. You should see the RealTimeIsUniversal key at the bottom of the list, similar to the following:

   ```
   HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\TimeZoneInformation
       Bias                            REG_DWORD     0x1e0
       DaylightBias                    REG_DWORD     0xffffffc4
       DaylightName                    REG_SZ        @tzres.dll,-211
       DaylightStart                   REG_BINARY    00000300020002000000000000000000
       StandardBias                    REG_DWORD     0x0
       StandardName                    REG_SZ        @tzres.dll,-212
       StandardStart                   REG_BINARY    00000B00010002000000000000000000
       TimeZoneKeyName                 REG_SZ        Pacific Standard Time
       DynamicDaylightTimeDisabled     REG_DWORD     0x0
       ActiveTimeBias                  REG_DWORD     0x1a4
       RealTimeIsUniversal             REG_DWORD     0x1
   ```

## Related Topics<a name="server-time-related-topics"></a>

For more information about how the Windows operating system coordinates and manages time, including the addition of a leap second, see the following documentation:

+ [How the Windows Time Service Works](https://docs.microsoft.com/en-us/windows-server/networking/windows-time-service/how-the-windows-time-service-works) \(Microsoft\)

+ [W32tm](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-xp/bb491016(v=technet.10)) \(Microsoft\)

+ [How the Windows Time service treats a leap second](https://support.microsoft.com/en-us/help/909614/how-the-windows-time-service-treats-a-leap-second) \(Microsoft\)

+ [The story around Leap Seconds and Windows: It's likely not Y2K](https://blogs.msdn.microsoft.com/mthree/2015/01/08/the-story-around-leap-seconds-and-windows-its-likely-not-y2k/) \(Microsoft\)