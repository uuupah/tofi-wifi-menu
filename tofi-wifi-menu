#!/usr/bin/env bash
# # based on https://github.com/ericmurphyxyz/rofi-wifi-menu/blob/master/rofi-wifi-menu.sh

# # ty to peter bill from https://unix.stackexchange.com/questions/789250/how-to-programmatically-kill-a-notify-send-window
notifid=$(notify-send -p "getting list of wifi networks...")

# Get a list of available wifi connections and morph it into a nice-looking list
wifi_list=$(nmcli --fields "SECURITY,SSID,BARS" device wifi list | sed 1d | sed -E 's/WPA*.?\S/[X] /g' | sed "s/^--/[-] /g" | sed "/--/d" | sed 's/\[X\]  \[X\]/[X]/g' | sed "s/\[X\] */[X] /g" | sed "s/\[-\] */[-] /g" | sed "s/ *$//" | awk '!seen[$0]++')

printf "$(printf "$wifi_list" | head -n 1 | awk '{print length($0); }')"

# +11 for the prompt, *8 for the character width, +10 for the padding and border on tofi
width=$(($(($(($(printf "$wifi_list" | head -n 1 | awk '{print length($0); }')+11))*8))+10))

connected=$(nmcli -fields WIFI g)
if [[ "$connected" =~ "enabled" ]]; then
	# toggle="󰖪  Disable Wi-Fi"
	toggle="disable wifi"
elif [[ "$connected" =~ "disabled" ]]; then
	# toggle="󰖩  Enable Wi-Fi"
	toggle="enable wifi"

fi

# remove the notification
notify-send -r $notifid -t 1 -u low "finished"

# Use tofi to select wifi network
chosen_network=$(echo -e "$toggle\n$wifi_list" | uniq -u | tofi --prompt-text "wifi ssid: " --width "$width")
# Get name of connection
read -r chosen_id <<< "${chosen_network:4:-5}"

if [ "$chosen_network" = "" ]; then
	exit
elif [ "$chosen_network" = "enable wifi" ]; then
	nmcli radio wifi on
elif [ "$chosen_network" = "disable wifi" ]; then
	nmcli radio wifi off
else
	# Message to show when connection is activated successfully
  	success_message="You are now connected to the Wi-Fi network \"$chosen_id\"."
	# Get saved connections
	saved_connections=$(nmcli -g NAME connection)
	if [[ $(echo "$saved_connections" | grep -w "$chosen_id") = "$chosen_id" ]]; then
		nmcli connection up id "$chosen_id" | grep "successfully" && notify-send "connection established" "$success_message"
	else
		if [[ "$chosen_network" =~ "[X]" ]]; then
			# # needing to call an empty echo is ugly, is there a way to get around it?
			wifi_password=$(echo "" | tofi --require-match=false --prompt-text "password: " --height 26 )
		fi
		nmcli device wifi connect "$chosen_id" password "$wifi_password" | grep "successfully" && notify-send "Connection Established" "$success_message"
    fi
fi
