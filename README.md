!pip install matplotlib seaborn
import subprocess
import platform
import re

def scan_wifi():
    os_type = platform.system()
    wifi_list = []

    try:
        if os_type == "Windows":
            result = subprocess.check_output("netsh wlan show networks mode=bssid", shell=True).decode()
            ssids = re.findall(r"SSID \d+ : (.+)", result)
            signals = re.findall(r"Signal\s+: (\d+)%", result)
            for ssid, signal in zip(ssids, signals):
                wifi_list.append((ssid.strip(), int(signal)))

        elif os_type == "Linux":
            result = subprocess.check_output("nmcli -f SSID,SIGNAL dev wifi", shell=True).decode()
            lines = result.strip().split("\n")[1:]
            for line in lines:
                parts = line.strip().split()
                if len(parts) >= 2:
                    ssid = " ".join(parts[:-1])
                    signal = int(parts[-1])
                    wifi_list.append((ssid.strip(), signal))
        else:
            print("Unsupported OS for WiFi scanning.")
    except Exception as e:
        print(f"Error: {e}")

    return wifi_list
import matplotlib.pyplot as plt
import seaborn as sns

def signal_to_color(signal):
    """Map signal strength to RGB color mood"""
    if signal > 75:
        return "#00bfff"  # calm blue
    elif signal > 50:
        return "#7fff00"  # fresh green
    elif signal > 25:
        return "#ffbf00"  # warm orange
    else:
        return "#ff4d4d"  # danger red

def generate_mood_visual(wifi_data):
    if not wifi_data:
        print("No WiFi data found.")
        return

    labels = [ssid for ssid, _ in wifi_data]
    signals = [signal for _, signal in wifi_data]
    colors = [signal_to_color(s) for s in signals]

    plt.figure(figsize=(10, 6))
    bars = plt.barh(labels, signals, color=colors)
    plt.xlabel("Signal Strength (%)")
    plt.title("WiFi Mood Light Mapper")
    plt.xlim(0, 100)

    for bar, signal in zip(bars, signals):
        plt.text(bar.get_width() + 1, bar.get_y() + 0.4, f"{signal}%", va='center')

    plt.tight_layout()
    plt.show()
wifi_data = scan_wifi()
if wifi_data:
    for ssid, signal in wifi_data:
        print(f"{ssid} - {signal}%")
    generate_mood_visual(wifi_data)
else:
    print("No WiFi networks found or insufficient permissions.")
    

