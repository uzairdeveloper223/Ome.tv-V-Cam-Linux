# Virtual Camera and Audio Setup for Linux

A lightweight utility for creating virtual camera and audio devices on Linux systems. This tool allows you to set up virtual webcam and audio sources that can be used with various video conferencing platforms.

## Prerequisites

- Linux operating system
- `v4l2loopback` kernel module
- `pactl` (PulseAudio)
- OBS Studio
- Root/sudo access

## Installation

Ensure you have the required dependencies installed:

```bash
sudo apt-get install v4l2loopback-dkms v4l2loopback-utils obs-studio
```

## Usage

### Virtual Camera Setup

First, remove any existing virtual camera devices:

```bash
sudo modprobe -r v4l2loopback
```

Create a new virtual camera with a realistic device label to avoid detection:

```bash
sudo modprobe v4l2loopback exclusive_caps=1 card_label="Integrated Camera (13d3:5415)" video_nr=10
```

The camera will be available at `/dev/video10` and will appear as a standard integrated camera to applications.

### Virtual Audio Setup

Create a virtual speaker output:

```bash
pactl load-module module-null-sink sink_name=RealtekSpeakers sink_properties=device.description="Realtek Speakers"
```

Create a virtual microphone input:

```bash
pactl load-module module-remap-source master=RealtekSpeakers.monitor source_name=VirtualMic source_properties=device.description="Built-In Audio Stereo"
```

### Removing Virtual Audio Devices

List all loaded audio modules:

```bash
pactl list modules short | grep -E "null-sink|remap-source"
```

This will display module IDs. Unload them using:

```bash
pactl unload-module <first_module_id> && pactl unload-module <second_module_id>
```

Replace `<first_module_id>` and `<second_module_id>` with the actual IDs from the previous command.

### Disabling Physical Camera

To disable your physical webcam:

```bash
sudo modprobe -r uvcvideo
```

To re-enable it:

```bash
sudo modprobe uvcvideo
```

## OBS Studio Setup

OBS Studio allows you to stream video files or any content to your virtual camera device.

### Initial Configuration

1. Launch OBS Studio from your applications menu or terminal:

```bash
obs
```

2. On first launch, the Auto-Configuration Wizard will appear. You can skip this or configure it for your needs.

### Adding Video Source

1. In the Sources panel at the bottom, click the plus icon
2. Select "Media Source" from the list
3. Give it a name like "Video Feed" and click OK
4. In the properties window:
   - Check "Local File"
   - Click Browse and select your video file
   - Check "Loop" if you want the video to repeat
   - Check "Restart playback when source becomes active"
   - Click OK

### Alternative Sources

You can add different types of sources depending on your needs:

- **VLC Video Source**: Better for playlists and multiple videos
- **Window Capture**: Share a specific window
- **Screen Capture**: Share your entire screen
- **Image**: Display a static image

### Starting Virtual Camera

1. Go to Tools menu at the top
2. Select "Start Virtual Camera"
3. OBS will now output to `/dev/video10`

If the virtual camera option is not available:

```bash
sudo apt-get install obs-v4l2sink
```

Or for newer OBS versions, the virtual camera is built-in and should work automatically with your v4l2loopback device.

### OBS Settings for Better Performance

1. Go to File > Settings
2. Under Output:
   - Set Output Mode to "Advanced"
   - Encoder: "x264"
   - Rate Control: "CBR"
   - Bitrate: 2500
3. Under Video:
   - Base Resolution: 1920x1080
   - Output Resolution: 1280x720
   - FPS: 30

### Audio Configuration in OBS

1. Go to Settings > Audio
2. Set your virtual microphone as the Mic/Auxiliary Audio device
3. Adjust audio levels using the mixer panel in the main window

## Using with Ome.tv

Once your virtual camera is running through OBS:

1. Open your web browser and navigate to Ome.tv
2. When prompted for camera access, allow it
3. Click on the camera settings or preview area
4. Look for a camera selection dropdown menu
5. Select "Integrated Camera (13d3:5415)" or whatever label you set
6. The video from OBS should now appear in your preview
7. If the camera doesn't switch automatically, refresh the page and select it again

### Browser Permissions

Make sure your browser has permission to access the camera:

**For Firefox:**
- Click the lock icon in the address bar
- Go to Permissions
- Ensure Camera is set to Allow

**For Chrome/Chromium:**
- Click the lock icon in the address bar
- Click Site Settings
- Set Camera to Allow

## How It Works

This setup creates virtual devices that mimic real hardware:

- The virtual camera uses `v4l2loopback` with a realistic device label
- OBS Studio outputs its video feed to the virtual camera device
- Virtual audio devices use PulseAudio's module system
- The `exclusive_caps=1` flag ensures proper compatibility with video applications

## Complete Workflow

Here is the complete process from start to finish:

```bash
# Step 1: Create virtual camera
sudo modprobe -r v4l2loopback
sudo modprobe v4l2loopback exclusive_caps=1 card_label="Integrated Camera (13d3:5415)" video_nr=10

# Step 2: Create virtual audio devices
pactl load-module module-null-sink sink_name=RealtekSpeakers sink_properties=device.description="Realtek Speakers"
pactl load-module module-remap-source master=RealtekSpeakers.monitor source_name=VirtualMic source_properties=device.description="Built-In Audio Stereo"

# Step 3: Disable physical camera (optional)
sudo modprobe -r uvcvideo

# Step 4: Launch OBS
obs

# Step 5: In OBS, add your media source and start virtual camera

# Step 6: Open Ome.tv and select the virtual camera
```

## Notes

- Module changes are temporary and will reset after system reboot
- Make sure to stop any applications using the camera before removing modules
- Video number can be changed from `10` to any available number
- Device descriptions can be customized to match your system
- OBS must be running with virtual camera active for the video to appear
- Keep OBS window open while using the virtual camera

## Troubleshooting

If the virtual camera is not detected:

```bash
v4l2-ctl --list-devices
```

If audio modules fail to load, check PulseAudio status:

```bash
pulseaudio --check
```

If OBS virtual camera is not working:

```bash
# Check if v4l2loopback is loaded
lsmod | grep v4l2loopback

# Verify the device exists
ls -l /dev/video*
```

If the browser cannot see the camera:

- Restart your browser completely
- Check browser permissions
- Try a different browser
- Make sure OBS virtual camera is started

## Making Setup Persistent

To make the virtual camera load automatically on boot, create a configuration file:

```bash
sudo nano /etc/modprobe.d/v4l2loopback.conf
```

Add this line:

```
options v4l2loopback exclusive_caps=1 card_label="Integrated Camera (13d3:5415)" video_nr=10
```

Then add the module to load on boot:

```bash
sudo nano /etc/modules-load.d/v4l2loopback.conf
```

Add:

```
v4l2loopback
```

## Author

Uzair Mughal

## License

This project is provided as-is for educational and legitimate purposes only.
