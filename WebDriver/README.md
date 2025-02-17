The WebDriver variant. Requires a driver program:
- `chromedriver` for Chromium. Should be packaged in your Linux distribution (for example, [`chromium-driver`](https://packages.debian.org/stable/chromium-driver) in Debian), otherwise is available [from Google](https://googlechromelabs.github.io/chrome-for-testing/).
- `geckodriver` for Firefox. Available [from Mozilla](https://github.com/mozilla/geckodriver/releases/) or may be packaged in your Linux distribution (for example, [`geckodriver`](https://archlinux.org/packages/extra/x86_64/geckodriver/) in Arch Linux).

Other WebDriver-capable browsers should work with minor tweaks of `class WebDriver`.
