# UI lab results

Tool parts that I chose: Part 3 Robot Framework and Part 4 Cypress

## Part 0
- Step 0.1, Chrome version: 157.0.8103.79 (Official Build) (arm64)
- Step 0.3, ChromeDriver version: ChromeDriver 157.0.8103.79
- Step 0.4, locator of Pizza, Please!: `button[name="pizza-please"]`
- Step 0.4, locator of Excluded tools: `select`
- Step 0.4, locator of Must be vegetarian: `#default-checkbox`
- Step 0.4, locator of Custom Pizza Name: `input#pizza-name`
- Step 0.5, pizza name, tool and calories of the run by hand:

| Step | Actual result | Pass or fail |
|------|---------------|--------------|
| 1    |               |              |
| 2    |               |              |
| 3    |               |              |
| 4    |               |              |
| 5    |               |              |
| 6    |               |              |
| 7    |               |              |

## Part 3: Robot Framework
- robot --version: Robot Framework 7.5.1 (Python 3.13.11 on darwin)
- Step 3.2, failure message for the wrong title: `Title should have been 'Web Form' but was 'Web form'.`
- Step 3.8, run 1, result, elapsed time and pizza name: PASS, 00:00:07.098, Lab Pizza 20261123-173831 (tool Scissors, 425 calories per slice)
- Step 3.8, run 2, result, elapsed time and pizza name: PASS, 00:00:06.917, Lab Pizza 20261123-173838 (tool Scissors, 475 calories per slice)

## Part 4: Cypress
- node --version, npm --version and npx cypress version: v24.12.0, 11.10.0, Cypress 16.2.1
- Step 4.3, failure message for the wrong title: `Timed out retrying after 10000ms`, with `-'Web form'` (actual) and `+'Web Form'` (expected)
- Step 4.6, tool and calories in the answer of POST /api/pizza: Pizza cutter, 475 (run 1)
- Step 4.7, the line of the admin list that shows your pizza: Lab Pizza 20261123-173912 (tool=Pizza cutter, ingredients_number=7)
- Step 4.8, run 1, result, duration and pizza name: All specs passed, 00:09 (QP-01 00:07), Lab Pizza 20261123-173912
- Step 4.8, run 2, result, duration and pizza name: All specs passed, 00:08 (QP-01 00:07), Lab Pizza 20261123-173926
- Step 4.9 (optional), browser and result: not done

## Part B
1. The three number boxes (Max Calories per Slice, Min and Max Number of Toppings) share the id `floating_filled`, and `$$("#floating_filled").length` returned 3, so the id cannot pick one box. The tests start from the label text instead: `//label[contains(.,'Max Calories')]/../input` in Robot and `cy.contains('label', 'Max Calories per Slice').siblings('input')` in Cypress. If QuickPizza gave each box a unique id such as `max-calories`, a test could use `#max-calories`. That helps a tester because a unique id stays the same when the label wording, translation or layout changes, and it is the shortest and least ambiguous locator.

2. It is a consistency oracle: the test does not know which tool is correct, it checks that two outputs of the same system agree, the recommendation on the home page and the admin list. It would catch a fault where the server stores the pizza with a different tool than the one it returned to the customer, for example the admin list showing `tool=Knife` after the page showed `Tool: Pizza cutter`. In my runs both views matched, e.g. Pizza cutter in both places for Lab Pizza 20261123-173912.

3. The page shows the name, dough, ingredients, tool and calories, but no vegetarian flag. The response of `POST /api/pizza` contains `"vegetarian": true`, so the Cypress test could add `expect(response.body.vegetarian).to.be.true` inside the `cy.get('@pizza')` block. A check on the page itself would need test data from another source, a list of non-vegetarian ingredients such as meat and fish, and would then check that no item in `#recommendations li` matches one of them.

4. Robot Framework needed explicit waits: `Wait Until Element Is Visible css:h1` for the heading, `Wait Until Page Contains Element` for the Knife option, and the `svelte-announcer` wait before the admin login. Cypress waited by itself, because every `cy.get` and `should` retries for 10 seconds, and only the admin login still needed the `#svelte-announcer` check. QP-01 was a little quicker to write in Cypress, and `cy.intercept` also let me check the request and the server answer. Robot's log.html was the most readable report: each keyword opens into its steps with values such as `${tool} = Scissors`, and each screenshot sits at the step that took it. Cypress gave a video and the Command Log, which show the page at every command. QP-01 took about 7 seconds in both tools.

5. I would use Cypress for Project 2. First, it needs no ChromeDriver: Robot needed a driver that matched Chrome 157 exactly and a CHROMEDRIVER variable, which breaks after every Chrome update, while Cypress ran right after `npm install`. Second, its automatic retry made QP-01 pass twice without hand-written waits, and `cy.intercept` plus the automatic video gave stronger checks and ready-made evidence.

6. (Part 3) The locator of the calories box lives only in `robot/resources/quickpizza.resource`, in the variable `${MAX CALORIES}`. If QuickPizza changed it, I would change that one line in the Variables section. `pizza_scenario.robot` only calls keywords such as `Fill In The Pizza Request    600    Knife    ${name}` and never names a locator, so the test file stays the same, and every test that uses the resource is repaired at once.

7. (Part 4) Cypress clears cookies, local storage and session storage before every test. QuickPizza runs on several servers, a cookie keeps one browser on one server, and the admin list shows only the pizzas of that server. If steps 6 and 7 were a second test, it would start without the cookie, could land on another server, and `cy.contains('li', pizzaName)` would time out because the pizza is not in that list. The second test would also lose `pizzaName` and the `@tool` alias, because aliases are reset between tests.
