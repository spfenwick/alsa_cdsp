# ALSA CamillaDSP "I/O" plugin - Really an "O" plugin as only Playback (output) is supported.
This is an ALSA I/O plugin for use with CamillaDSP for audio playback.  It starts a CamillaDSP process and streams data to it via a pipe.  To playback programs it responds like a normal ALSA device.  The actual output device is whatever you configure in the CamillaDSP YAML configuration file.

This is a trimmed down version of the original plugin by scripple.  This version is intended to work with CamillaDSP v2 statefiles:

* It removes the ability to modify the CamillaDSP config file as CamillaDSP now supports overriding key parameters via the command line.

* It removes the ability to specify a config file on the CamillaDSP on the assumption that the config file is contained in the statefile.

* It removes the ability to specify extra_samples as this is specfied of the CamillaDSP config file and is automatically adjusted if the sample rate is overridden.

* It removes the vol_file option as this functionality is now provides by the statefile.

queuelimit: 1 is highly recommended in the CamillaDSP config file or else you will experience large latency in the audio responding to user input.  It's also suggested you set the playback format to the largest bit depth your hardware can handle.  The playback device does not have to use {channels} if you are using CamillaDSP to change the number of output channels.

Here is a sample .asoundrc (or /etc/asound.conf) file.

<pre>
# This declares an ALSA device that you can specify to playback programs.
# You can use any name you wish that ALSA supports not just camilladsp.
# To use it you specify it to alsa like "aplay -D camilladsp"
pcm.camilladsp {
    # type cdsp must be present - it declares to use this plugin.
    # The type is NOT a variable name you can choose.
    # You can however create multiple type cdsp plugins with different names
    # if you want to specify different parameters selected by specifying a
    # different ALSA device.
    type cdsp
    
    #######################################################################
    #######################################################################
    # Required parameters.
    # The parameters in this section must be specified as a valid set.
    #######################################################################
    #######################################################################
      
    # cpath specifies the absolute path to the CamillaDSP executable.
    # CamillaDSP must have executable permission for any user that runs an
    # audio program that uses this plugin.
    cpath "/path/to/camilladsp"

    #######################################################################
    # Capability Enumeration
    #
    # The plugin will announce the hw_params it supports based on the 
    # following settings.  Channels and sample rates must be specified.
    # The plugin will automatically enumerate all the formats that 
    # CamillaDSP supports.
    #######################################################################
    # Channels can be specified as a single value like this.
    channels 2
    # Or a range like this. (Uncomment the lines.)
    #min_channels 1
    #max_channels 2
    # But only one method can be used.
      
    # Sample rates can be configured as a specific list like this.  
    # (Up to 100 entries.)
    rates = [
      44100 
      48000 
      88200 
      96000
      176400
      192000
      352800
      384000
    ]
    # Or as a range like this.  (Uncomment the lines.)  
    #min_rate 44100
    #max_rate 384000      
    # Note that if you use a range like this ALSA will accept ANYTHING in
    # that range.  Even something odd like 45873.  If you aren't using
    # CamillaDSP's resampler and don't have a very unusual DAC you are
    # probably better off with the list format above.
    #######################################################################
    # End Capability Enumeration
    #######################################################################
      
    #######################################################################
    #######################################################################
    # End Required Parameters
    #######################################################################
    #######################################################################      

    #######################################################################
    #######################################################################
    # Optional Parameters
    #######################################################################
    ####################################################################### 
      
    # If you wish to specify additional arguments to CamillaDSP you can
    # specify them using the cargs array.  Numeric arguments must be quoted
    # in strings or the plugin will fail to load. You should not specify
    # the hw_params arguments here as the plugin will take care of that as
    # they change.
    # Note that since a CamillaDSP config file is no longer supplied the
    # -s parameter is required in order for CamillaDSP to start.
    cargs [
      -s "/path/to/statefile.yml"
      -p "1234"
      -a "0.0.0.0"
      -l warn
    ]      
      
    # An option not directly related to camilladsp.  A command to run when
    # the plugin is first created.  (Meaning when a playback program opens
    # the audio device.)  Use it for whatever you want.  Set gpio pins on a 
    # raspberry pi to turn on your amp for example.
    start_cmd "/path/to/start_command"

    # An option not directly related to camilladsp.  A command to run when
    # the CamillaDSP process is closed.  (Meaning when a playback program
    # closes the audio device or changes the HW params.)  Use it for
    # whatever you want.  A good example is storing the gain and mute
    # settings of CamillaDSP for use with the vol_file option below.
    #
    # Note this command is called just before the CamillaDSP process is
    # closed. That is not necessarily when you are done listening to music.
    # So unlike the start_cmd this might not work well as a signal to turn
    # off your amp.  Also note that CamillaDSP is still expecting audio
    # while this command is running so it may cause a harmless warning
    # about a buffer underrun to be emitted by CamillaDSP if you have
    # CamillaDSP's log level set sufficiently high.
    camilla_exit_cmd "/path/to/camilla_exit_command"
}
</pre>

To build this plugin you need the standard build utilities (gcc) and the ALSA development package.
On a debian based system the ALSA development package is installed using the following command.

<pre>
sudo apt install libasound2-dev
</pre>

Then just build like most things.

<pre>
make
sudo make install
</pre>

Place an .asoundrc file like the one above in your home directory (for a single user) or in /etc/asound.conf for system wide definitions.

Then point your audio programs to use "camilladsp" or whatever you named your plugin declaration as the ALSA device.

Also, please note that this plugin runs CamillaDSP directly on its own.  You should not have CamillaDSP running as a service like described in the CamillaDSP documentation when using this plugin.  They might fight over the same audio output hardware.
