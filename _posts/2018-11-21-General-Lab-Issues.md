

### Error while deploying CSR 1000v directly on  ESXi

### A Required Disk Image Was Missing

```sh
$ ovftool --name="CSR1"  --acceptAllEulas -ds="datastore1" --net:"GigabitEthernet1"="VM Network"  --net:"GigabitEthernet2"="VM Network" --net:"GigabitEthernet3"="VM Network"  /filepath/csr1000v-universalk9.16.03.06\ \(1\)\ copy.ova vi://192.168.1.13/
Opening OVA source: /filepath/Downloads/csr1000v-universalk9.16.03.06 (1) copy.ova
The manifest validates
Enter login information for target vi://192.168.1.13/
Username: root
Password: *********
Opening VI target: vi://root@192.168.1.13:443/
Deploying to VI: vi://root@192.168.1.13:443/
Disk progress: 4%

ovftool --name="KISE"  --acceptAllEulas -ds="datastore1" --net:"GigabitEthernet1"="VM Network"  --net:"GigabitEthernet2"="VM Network" --net:"GigabitEthernet3"="VM Network"  /filepath/csr1000v-universalk9.16.03.06\ \(1\)\ copy.ova vi://192.168.1.13/

Opening OVA source: /filepath/Downloads/csr1000v-universalk9.16.03.06 (1) copy.ova
The manifest validates
Enter login information for target vi://192.168.1.13/
Username: root
Password: *********
Opening VI target: vi://root@192.168.1.13:443/
Deploying to VI: vi://root@192.168.1.13:443/
Disk progress: 4%
```


```sh
./ovftool --noSSLVerify \
        --name="KISE"  \
        --acceptAllEulas \
        -ds="datastore1" \
        --diskMode=thick \
        --network="VM Network"  \
        /Users/vikassri/Downloads/ISE-2.1.0.474-mini.ova \
        vi://192.168.1.13/

./ovftool --noSSLVerify \
          --name="KISE"  \
          --acceptAllEulas \
          -ds="datastore1" \
          --diskMode=thin \
          --network="VM Network"  \
          /Users/vikassri/Downloads/Cisco_Firepower_Management_Center_Virtual_VMware-6.2.3-83/Cisco_Firepower_Management_Center_Virtual_VMware-ESXi-6.2.3-83.ovf \
          vi://192.168.1.13/



qemu-img convert -c -O qcow2 /opt/unetlab/tmp/0/c06281a8-161c-4aa5-aea4-4d304b13b6d4/1/megasasa.qcow2 /opt/unetlab/addons/qemu/vwlc-8.7.102/megasasa.qcow2


qemu-img convert -c -O qcow2 /opt/unetlab/tmp/0/c06281a8-161c-4aa5-aea4-4d304b13b6d4/1/virtioa.qcow2  /opt/unetlab/addons/qemu/win-7test/virtioa.qcow2
```

##### Ovftool with thin provision

./ovftool --acceptAllEulas **-dm=thin**  -ds="datastore1" --net:"External"="VM Network" --net:"Internal"="VM Network" --net:"Management"="VM Network" Cisco_Firepower_NGIPSv_VMware-ESXi-6.2.0-362.ovf  vi://192.168.1.13/


##### Downloading from Datastore to local machine

wget --no-check-certificate --http-user=root --http-password=********** 'https://192.168.1.13/folder/OS_ISOs/W10X64.1709.ENU.DEC2017.iso?dcPath=ha%2ddatacenter&dsName=datastore1'  -O  Windows.iso



ESXi

https://talesfromthedatacenter.com/2016/02/esxi-6-install-stuck-on-relocating-modules-and-starting-up-the-kernel/


### Re-arranging iTerm

Thanks to `http://azaleasays.com/2014/03/05/iterm2-merge-a-pane-back-to-window-or-tab-bar/`

The secret key combination to drag a pane back to a window or to the tab bar is: `command + shift + option`, while dragging the pane body (not its header!). View the screencast below and you’ll figure it out:


### Having iTerm handle the telnet:// URL

![](/assets/markdown-img-paste-20180528184424236.png)


**ASA in Eve-NG**

![](/assets/markdown-img-paste-20180703130517336.png)


**Java Issues**

On MAC Look for JavaAppletPlugin.plugin , Open in Atom and In tree view do the following:
Comment “jdk.jar.disabledAlgorithms” in the file of”lib/security/java.security”


**Adding Capability to Upgrade in EVE VM**

root@vagrant-eve:~# cat /etc/apt/sources.list | grep eve
deb [arch=amd64] http://www.eve-ng.net/repo xenial main
# deb-src [arch=amd64] http://www.eve-ng.net/repo xenial main

Delete or comment  the refrence to eve repo .

**X Window Forwarding**
ssh -Y vikassri@192.168.1.195

**TMUX Integration with iTerm**

`tmux` should be installed in the remote machine where we are sshing to.
Once Connected to the remote machine

`tmux -CC` to start the tmux session

`Cmnd + n` for New window
`Cmnd + Shift + Option` To rearragne the windows
`Esc` to dettach in the main window
`tmux -CC attach` to attach back the same arrangment.

![](assets/markdown-img-paste-20191117154150432.png)
![](assets/markdown-img-paste-20191117154337895.png)


# EVE Issues

/opt/unetlab/wrappers/unl_wrapper -a stopall
then run again
apt update
apt install --reinstall eve-ng-pro





<!DOCTYPE html>
<html>
<head>
<meta name="viewport" content="width=device-width, initial-scale=1">
<style>
@keyframes fade-in-up {
	 0% {
		 opacity: 0;
	}
	 100% {
		 transform: translateY(0);
		 opacity: 1;
	}
}

.video iframe {
	 max-width: 100%;
	 max-height: 100%;
}

.video.stuck {
	 position: fixed;
	 bottom: 20px;
	 right: 20px;
	 width: 260px;
	 height: 145px;
	 transform: translateY(100%);
	 animation: fade-in-up 0.75s ease forwards;
	z-index: 1;
}
</style>

<div class="video-wrap">
  <div class="video">
    <iframe width="600" height="340" src="https://www.youtube.com/embed/0pThnRneDjw" frameborder="0" gesture="media" allowfullscreen></iframe>
  </div>
</div>


<script type="text/javascript">
(function($) {
	var $window = $(window);
	var $videoWrap = $('.video-wrap');
	var $video = $('.video');
	var videoHeight = $video.outerHeight();

	$window.on('scroll',  function() {
		var windowScrollTop = $window.scrollTop();
		var videoBottom = videoHeight + $videoWrap.offset().top;

		if (windowScrollTop > videoBottom) {
			$videoWrap.height(videoHeight);
			$video.addClass('stuck');
		} else {
			$videoWrap.height('auto');
			$video.removeClass('stuck');
		}
	});
}(jQuery));
</script>






</body>
</html>
