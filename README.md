# Paulie Pomodoro

A beautiful, theme-aware Pomodoro timer for VS Code, designed to help you stay focused and productive. "Hey Paulie! Paulie. More wine."

## Installation

Install this extension from the [VS Code marketplace](https://marketplace.visualstudio.com/items?itemName=DarrenJaworski.paulie-pomodoro).

OR

With VS Code open, search for `paulie-pomodoro` in the extension panel (`Ctrl+Shift+X` on Windows/Linux or `Cmd(⌘)+Shift+X` on MacOS) and click install.

OR

With VS Code open, launch VS Code Quick Open (`Ctrl+P` on Windows/Linux or `Cmd(⌘)+P` on MacOS), paste the following command, and press enter.

`ext install darrenjaworski.paulie-pomodoro`

## Features

- **Pomodoro Timer**: Start, pause, reset, and switch between work and break sessions directly from the Activity Bar or the webview panel.
- **Status Bar Integration**: See your timer and session type at a glance in the VS Code status bar. Click to start/pause and open the controls.
- **Webview Panel**: Modern, responsive UI with large timer display, session type, and intuitive controls.
- **Theme Awareness**: All UI elements (timer, buttons, footer) adapt to your VS Code color theme, both light and dark.
- **Customizable Durations**: Configure your work and break session lengths in the extension settings.
- **Optional Notifications**: Enable/disable notifications when sessions end.
- **Automatically switch session**: Automatically switch sessions and leave timer running.

![Preview](https://raw.githubusercontent.com/darrenjaworski/paulie-pomodoro/refs/heads/main/paulie-pomodoro-preview.png)

## Requirements

No special requirements. Just install and start using!

## Extension Settings

This extension contributes the following settings:

- `paulie-pomodoro.workingSessionLength`: Length of the working session in minutes (default: 24)
- `paulie-pomodoro.breakSessionLength`: Length of the break session in minutes (default: 6)
- `paulie-pomodoro.enableNotifications`: Enable info notifications (default: disabled)
- `paulie-pomodoro.autoSwitchSessions`: Enable automatically switching the session and leaving the timer running (default: disabled)

## Commands

- **Paulie Pomodoro: Start Timer**: Start the Pomodoro timer
- **Paulie Pomodoro: Pause Timer**: Pause the timer
- **Paulie Pomodoro: Reset Timer**: Reset the timer to the beginning of the current session
- **Paulie Pomodoro: Switch Session Type**: Switch between work and break sessions

You can access these commands from the Command Palette or via the webview controls.

## Known Issues

[Please report any bugs or issues on the extension's Github repo.](https://github.com/darrenjaworski/paulie-pomodoro/issues/new)

## Release Notes

### 1.0.7

- chore: update dev dependencies to patch security advisories

### 1.0.6

- chore: update readme

### 1.0.5

- feat: fluid session setting

### 1.0.4

- fix: extra word in footer

### 1.0.3

- chore: update readme with new notification setting
- feat: enable notification setting

### 1.0.2

- feat: add notification setting, set to false
- refactor: removed unused code

### 1.0.1

- fix: webview header title
- chore: readme updates

### 1.0.0

- feat: Pomodoro timer
- feat: webview controls
- feat: status bar integration
- feat: theme support
- feat: settings
- feat: commands
