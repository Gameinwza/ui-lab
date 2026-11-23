# ui-lab

Automated UI tests for ITDS362. Tool parts: **Part 3 Robot Framework** and **Part 4 Cypress**.

## robot/
Robot Framework + SeleniumLibrary tests: the web form (`first.robot`, `web_form.robot`) and QP-01 (`pizza_scenario.robot`), with the QuickPizza keywords in `resources/quickpizza.resource`. Needs the `CHROMEDRIVER` variable from Part 0.

```
cd robot && source .venv/bin/activate && robot -d results/run1 -v BROWSER:headlesschrome pizza_scenario.robot
```

## cypress/
Cypress specs for the web form and QP-01 in `cypress/e2e`, the `adminLogin` command in `cypress/support/commands.js`, plus the videos and screenshots of a full run.

```
cd cypress && npm install && npx cypress run --browser chrome
```
