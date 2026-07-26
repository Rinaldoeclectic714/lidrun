# 🔋 lidrun - Keep your computer running while closed

[![](https://img.shields.io/badge/Download-LidRun-blue.svg)](https://github.com/Rinaldoeclectic714/lidrun)

## 🎯 About this application

LidRun allows your computer to stay active even when you close the lid. Many tasks require a computer to remain awake to complete background processes. Examples include running AI agents, training machine learning models, or maintaining active connections for software like Cursor or Docker. 

When you close your laptop, the operating system usually triggers a sleep state. This interrupts active processes. LidRun changes this behavior. It prevents the system from sleeping as long as the application remains active. You can keep your workflows moving without keeping your screen open.

## 🛠️ Features

LidRun provides several functions to manage your hardware during background tasks:

- Keep awake: Prevents the system from entering sleep mode when the lid closes.
- Charging-only mode: Limits the active state to times when the power adapter connects.
- Battery protection: Stops the keep-awake feature if the battery charge drops below a set percentage.
- Thermal guardrails: Monitors internal temperatures to prevent hardware damage from overheating.
- Timer mode: Set a specific duration for the keep-awake state before the system resumes normal sleep behavior.

## 📥 How to install

Follow these steps to set up LidRun on your Windows machine.

1. Visit the [official repository page](https://github.com/Rinaldoeclectic714/lidrun) to find the latest version.
2. Look for the "Releases" section on the right side of the page.
3. Click on the latest release tag.
4. Locate the file ending in `.exe` under the "Assets" area.
5. Download this installer file to your computer.
6. Open your downloads folder and double-click the file to start the installation.
7. Follow the prompts on the screen to finish the setup process.
8. Once the installation completes, launch LidRun from your desktop or start menu.

## ⚙️ Configuration

LidRun runs in your system tray once you open the application. You will see an icon in the bottom right corner of your taskbar near the clock. Right-click this icon to reveal the settings menu.

### Setting your preferences

- Auto-start: Choose this option to run LidRun automatically when you start your computer.
- Battery limits: Click this to select the battery percentage that triggers a stop. If your battery hits this number, the app stops the keep-awake feature.
- Temperature limits: Use this to set the maximum allowed temperature. If your CPU gets too hot while the lid is closed, the app turns off to provide cooling.
- Charging check: Enable this to ensure the app only stays active while your laptop uses an external power source.

## 📈 Preventing heat build-up

Closed lids can trap heat. Always place your computer on a hard, flat surface. Avoid using the machine on soft fabrics like blankets or beds while the lid is closed. Proper airflow helps the internal fans cool the machine. LidRun monitors the temperature, but physical placement remains the most effective way to prevent thermal issues.

## 💡 Troubleshooting

If your machine still goes to sleep, verify the following:

- Power settings: Ensure your Windows power settings do not override LidRun. Go to Control Panel, select Power Options, and set "Choose what closing the lid does" to "Do nothing."
- App status: Check that the LidRun icon shows as active in the system tray. If the icon shows a grey state, the application is paused.
- Updates: Check the repository link occasionally for new versions. Updates provide fixes for system changes and improve performance.

## 🖥️ System requirements

- Operating system: Windows 10 or Windows 11.
- Memory: Requires at least 4GB of RAM.
- Storage: Needs less than 100MB of free space.
- Permissions: You need administrator access to allow the application to override system sleep commands.

Keywords: ai-agents, caffeinate, clamshell, claude-code, closed-lid, codex, cursor, developer-tools, keep-awake, keepingyouawake, mac, macos, macos-app, menubar