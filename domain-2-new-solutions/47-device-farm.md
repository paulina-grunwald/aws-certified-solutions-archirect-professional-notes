# AWS Device Farm

> **Cloud-based mobile + web app testing on real physical devices and desktop browsers. Run automated test suites (Appium, XCUITest, Espresso, Selenium) or interactive remote-control sessions on 100+ iOS / Android devices and Chrome / Firefox / Edge browsers. Use for QA + regression testing on real hardware (battery state, gestures, screen sizes) without owning a device lab. Lower SAP-C02 frequency but appears as the "test on real mobile devices in the cloud" answer.**

Maps to: **Domain 3.4 — Improve deployments**, **Domain 4.3 — Modernization**

---

## Overview

- **Real physical devices** in AWS data centers — iOS + Android phones & tablets
- **Desktop browsers** for web app testing
- Run **automated tests** or **interactive sessions** (remote control)
- Supported frameworks: **Appium (Java/Python/Node), XCUITest, Espresso, Calabash, Robotium, Selenium**
- **Parallel execution** across multiple devices
- Logs, screenshots, video recording, performance metrics

## Two Project Types

### Mobile App Testing
- Upload `.apk` (Android) or `.ipa` (iOS)
- Choose devices (specific or pool)
- Choose test framework + upload test package
- Or interactive remote session
- Get results + videos + logs

### Desktop Browser Testing
- Selenium-based
- Multiple browsers + versions
- Test scaling / responsive design
- Run from CI/CD

## Device Selection

- **Specific devices**: pick iPhone 15 Pro, Pixel 8, etc.
- **Device pools**: groups (e.g., "all iPhones", "Android 13+")
- **Private devices**: dedicated to your account (paid extra) — reserved access, no queue
- **Public devices**: shared with other AWS customers

## Test Frameworks

### Android
- Appium (Java, Python, Ruby, Node)
- Calabash
- Espresso
- Instrumentation
- Robotium
- UI Automator

### iOS
- Appium
- Calabash
- XCTest / XCUITest
- UI Automation (deprecated by Apple)

### Web
- Selenium WebDriver

### Built-in tests
- Built-in fuzz testing (random taps)
- No-code option for sanity testing

## Test Results

- **Pass / fail** per test
- **Video recording** of test session
- **Logs** (app + device system)
- **Screenshots** at each step
- **Performance**: CPU, memory, network, battery
- Export to S3 for archival

## Integration with CI/CD

- **AWS CodePipeline** — Device Farm as test stage
- **GitHub Actions / Jenkins / GitLab CI** — via AWS CLI
- **Failure** in Device Farm = pipeline fails

## Use Cases

- **Regression testing** on real devices pre-release
- **Mobile app QA** without buying / maintaining a device lab
- **Cross-browser web testing** at scale
- **Performance testing** on low-end devices (e.g., low-RAM Android)
- **Beta testing** with remote device access

## Device Farm vs Manual QA Lab

| | Device Farm | Self-Hosted Lab |
|---|---|---|
| Device coverage | 100s available | Limited by budget |
| Device freshness | AWS rotates | Manual upgrades |
| Geographic distribution | Multi-region | Single location |
| Parallelization | Easy | Hardware-limited |
| Cost | Per device-minute | Capex + maintenance |
| Best for | Variable test load | Heavy continuous testing |

## Device Farm vs Browser-Based (Synthetics)

- **CloudWatch Synthetics** = scheduled production monitoring with Puppeteer / Selenium
- **Device Farm** = pre-release QA on real mobile devices + desktop browsers
- Don't conflate — different lifecycle stages

## Pricing

- **Mobile**: $0.17 per device-minute (public)
- **Web**: $0.17 per browser-minute
- **Private device**: $250/month per device (dedicated, no queuing)
- **Unmetered devices** — flat $250/month for unlimited testing on one device slot
- Free tier: 1,000 device-minutes

## Common Patterns

### Pre-release regression
- Nightly CI: deploy beta build → Device Farm regression suite on top 20 devices
- Failure blocks release

### Remote interactive testing
- QA engineer requests device via console
- Browser-based remote control of physical phone
- Test gestures, sensors, network conditions

### Cross-version Android testing
- Test on Android 11, 12, 13, 14 in parallel
- Catch OS-specific regressions

### A/B variant validation
- Run new UI variant on 5 devices
- Capture screenshots / videos for design review

## Limitations

- **No camera input** simulation (Device Farm provides preset videos)
- **No SIM card** — cellular tests limited
- **Some emulators not supported** — focus is on real devices
- **Latency** in remote sessions (~100ms)
- **App must be uploadable** — protected APKs may not work

## Exam Tips

- "Test mobile app on real physical devices in the cloud" → **AWS Device Farm**
- "Cross-browser web testing automation" → **Device Farm desktop browsers**
- "100+ iOS / Android devices accessible via API"
- Supports **Appium / XCUITest / Espresso / Selenium**
- Integrates with **CodePipeline / GitHub Actions**
- Free tier: 1,000 device-minutes

## Exam Traps

- **Device Farm ≠ CloudWatch Synthetics** — Device Farm is pre-release on physical devices; Synthetics is production monitoring
- **Device Farm ≠ App Mesh / X-Ray** — those are runtime; Device Farm is QA
- **Real device queues can be slow** — popular devices have wait time; use private devices for guaranteed access
- **Private device costs $250/month per slot** — only economical for heavy continuous use
- **No camera / cellular input** — Device Farm is limited for hardware-dependent tests
- **Lower SAP-C02 frequency** — but tested as "real mobile device testing in cloud"
- **Selenium-based desktop tests run in container** — not real desktops
- **Test artifacts retained 30 days** — export to S3 for longer
