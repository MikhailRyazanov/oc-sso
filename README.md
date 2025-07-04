# oc-sso

Python wrapper around [OpenConnect](https://www.infradead.org/openconnect/) for [SAML 2.0](https://en.wikipedia.org/wiki/SAML_2.0) (SSO) web-based authentication to Cisco VPNs (open-source replacement for Cisco AnyConnect / Secure Client).

This is a minimalistic rewrite of [openconnect-sso](https://github.com/vlaci/openconnect-sso) avoiding as many dependencies as possible and, in particular, using a regular web browser (like Chromium of Firefox) instead of Qt WebEngine. The script is a single executable file and comes in 3 variants available in corresponding subdirectories:
- **Chromium**
  Using [Chromium](https://www.chromium.org/Home/) browser through the [Chrome DevTools protocol](https://chromedevtools.github.io/devtools-protocol/). Requires `websockets` ([websocket-client](https://pypi.org/project/websocket-client/)) Python package.
- **Firefox**
  Using [Firefox](https://www.mozilla.org/firefox/) browser through the [Marionette protocol](https://firefox-source-docs.mozilla.org/testing/marionette/index.html). Has no requirements besides the Python standard library.
- **WebDriver**
  Using Chromium or Firefox and potentially any other browser supporting the [WebDriver protocol](https://w3c.github.io/webdriver/). Requires a corresponding driver: `chromedriver` for Chromium or `geckodriver` for Firefox.


## Limitations

Only basic functionality is implemented.
- Intended only for Linux. Might work in other Unix-like systems but unlikely in Windows.
- No configuration files. If needed, make a shell wrapper passing the necessary command-line options.
- Groups and proxies are not supported. Only because I can’t test them.
- No automatic login. Should be doable like in `openconnect-sso` but is out of scope.


## Installation and usage

Simply download the desired variant in a directory from which it can be executed and make sure that it has the execute permission (or run using `python3 oc-sso ...`). Running `oc-sso -h` prints the available options:
```
  -L LEVEL, --log-level LEVEL
                        verbosity level for logging messages (ERROR, WARNING,
                        INFO, DEBUG; default: INFO)
  --version-string VERSION
                        AnyConnect version for authentication and OpenConnect
                        (default: 5.1.4.74)
  --sso-timeout SSO_TIMEOUT
                        timeout in seconds for web-browser SSO authentication
                        (default: 120)
  --attempts ATTEMPTS   number of attempts to communicate with the server at
                        the stages that can fail randomly (default: 5)
```
(The last one is needed because the server sometimes fails to redirect to the actual login page or to accept correct authorization responses immediately, but does so after repeated attempts; see “[Bugs](#bugs-and-suggestions)” below.)

The WebDriver variant has an additional option to select which driver (and thus browser) to use:
```
  --webdriver {chromedriver,geckodriver}
                        WebDriver implementation to use: chromedriver for
                        Chrome/Chromium or geckodriver for Firefox (default is
                        to try both)
```
All unrecognized options are passed to `openconnect`, which is launched after finishing the authentication. In particular, using `oc-sso -b vpn.server.name` will start `openconnect` in background and return.


## Security

A clean temporary browser profile is used every time. This means that your browser data cannot leak to the authentication server. But also that authentication cookies are not persistent, so functionality like “remember this device” will not work, and you’ll need to complete the whole process every time (persistence could be achieved by specifying a permanent browser profile in `NewSession()` but is out of scope). The temporary profile is created in `/tmp` with read/write permission only for the user and is deleted immediately after finishing the web authentication (before launching `openconnect`). If something goes very wrong, the temporary profile might remain, maybe even with the cookies, so keep an eye on it if this matters.

The original `openconnect-sso` is passing `--servercert` from the last authentication response to `openconnect`, but here this code is commented out as useless: everything is done through HTTPS anyway, and if you worry about spoofing, then certificates must be checked at *every* step instead of sending your credentials to an unconfirmed server and trusting the fingerprint received from it.

“OpenConnect needs to be run as the root user in order to make changes to the system networking configuration”, so it is launched using `sudo`, which will ask for your local password. If you want to allow this without requiring the password, add `NOPASSWD: /usr/sbin/openconnect` to your user in [sudoers](https://man7.org/linux/man-pages/man5/sudoers.5.html#SUDOERS_FILE_FORMAT) (as a precaution, this permission can be limited to executing `openconnect` only with specific command-line arguments, which can be found by running `oc-sso` with `--log-level DEBUG` or from <code>[ps](https://man7.org/linux/man-pages/man1/ps.1.html) -C openconnect -o args -ww</code>). Alternatively, see [“Running as non-root user”](https://www.infradead.org/openconnect/nonroot.html) in the OpenConnect documentation and modify the script accordingly (specifically, the `'sudo', 'openconnect',` line).


## Bugs and suggestions

Please open an issue if you notice something obviously wrong or know what causes the following problems and how to solve them properly (instead of circumventing with the `--attempts` option):
1. The authentication URL received from pre-authentication sometimes doesn’t redirect to the actual login page. Currently the script detects this state and retries pre-authentication (simply reloading the web page doesn’t help).
2. The final authentication response sometimes returns an error “The configuration of the VPN Server has changed. Please try again.”, however, resending the same data usually results in successful authentication.
3. Sometimes `openconnect` exits, complaining that the server rejected the authentication cookie. Retrying with the same cookie usually helps.

Otherwise, this repository serves mostly as a proof of concept, and no enhancements are planned. Thus, for any additional functionality, feel free to create your own fork. However, even better would be to work on porting this approach to OpenConnect itself, either as a built-in handler or as an “external-browser” script callable by `openconnect` between pre- and post-authentication (perhaps the already open OpenConnect [issue #425](https://gitlab.com/openconnect/openconnect/-/issues/425) is relevant).
