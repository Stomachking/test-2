# Outlook Takeaways Quiz

Build notes for the AEM team.

`index.html` is a self-contained prototype of a quiz component for AEM Sites. A visitor answers five multiple-choice questions. Each answer unlocks one key takeaway from the market outlook. After the fifth answer, a result screen shows the score and all five takeaways, and a short form has a link to the executive summary emailed to the visitor.

Open `index.html` in a browser to run it. There is no build step, no framework and no backend.

## Status

This is a prototype for stakeholder review, not production code.

- **Copy is draft.** The questions, answer options and explanations were written from five headlines only. The three "forces" in question 1 are placeholders. Replace everything with text from the outlook before launch.
- **Colours and typefaces are not the brand's.** They are set as CSS variables so they can be swapped (see [Theming](#theming)).
- **The form sends nothing and saves nothing.** It validates in the browser and then shows the "check your inbox" confirmation.
- **No email is sent.** "Send it again" only shows a note explaining what would happen in production.

## How it works

| Step | What the visitor sees |
| --- | --- |
| 1. Question | Heading, intro and an outlined "Your takeaways" panel on the left, with a running count ("1 of 5 unlocked"). One question with two to four answer rows on the right. Locked takeaways read "Takeaway 3 · Locked" with a padlock; the row for the current question is highlighted. The executive summary is listed last, locked. |
| 2. Answer | The chosen row is marked right or wrong and the correct row is marked. A sheet appears with the takeaway headline and explanation. The headline also replaces the locked label in the panel. |
| 3. Next | "Next question" moves on. Answered questions can be reopened from the panel. Questions must be answered in order. |
| 4. Result | After the last answer, "See your result" opens a full-width screen: score, a result message, the takeaways in columns, and the summary form. The left-hand heading and panel are hidden on this screen. |
| 5. Review | "Review" under a takeaway reopens that question. A "Back to your result" link and button return to the result screen. |
| 6. Check your inbox | Sending a valid form replaces it with a confirmation that names the address the link was sent to. Two actions sit underneath: "Send it again" and "Use a different email", which returns to the form with the details still filled in. The summary link itself is never shown on the page. |
| 7. Start again | Clears the answers. Form details already entered are kept. |

Result message thresholds: all correct shows the "high" message; at least half correct (rounded up) shows "mid"; anything lower shows "low".

## Content model

All text comes from one `CONFIG` object at the top of the script in `index.html`. It has the shape a Sling Model would export as JSON; an Edge Delivery block could read the same fields from its table.

| Dialog field | Type | `CONFIG` property | Where it shows |
| --- | --- | --- | --- |
| Eyebrow | Text | `eyebrow` | Small label above the heading, on the result screen and on the report cover |
| Title | Text | `title` | Main heading |
| Title emphasis | Text | `titleEmphasis` | Second part of the heading, set in italic |
| Intro | Text | `intro` | Line under the heading |
| Questions | Multifield | `questions[]` | One item per takeaway. Five here; the layout holds three to seven |
| Question | Text | `questions[].question` | Heading above the answer rows |
| Options | Nested multifield | `questions[].options[]` | Two to four answer rows |
| Correct option | Select | `questions[].correct` | Zero-based index of the right option |
| Takeaway headline | Text | `questions[].takeaway` | Reveal sheet, contents list and result columns |
| Takeaway explanation | Rich text | `questions[].explanation` | Reveal sheet and result columns |
| Result messages | Text | `results.high`, `.mid`, `.low` (each `title`, `text`) | Heading and line beside the score |
| Summary gate heading | Text | `gate.heading` | Sheet on the result screen, beside the form |
| Summary gate intro | Text | `gate.intro` | Under the gate heading |
| Submit label | Text | `gate.submitLabel` | Form button |
| Opt-in text | Rich text | `gate.optIn` | Checkbox label |
| Privacy text | Rich text | `gate.privacy` | Small print in the form |
| Confirmation heading | Text | `gate.thanks` | Shown in place of the form after sending, after "Thanks, [first name]." |
| Confirmation text | Text | `gate.sentText` | Under the confirmation heading. `{email}` is replaced with the address entered |
| Resend label | Text | `gate.resendLabel` | "Send it again" button |
| Change-email label | Text | `gate.changeLabel` | "Use a different email" link |
| Disclaimer | Rich text | `disclaimer` | Small print at the foot of the component |

The form fields themselves (first name, last name, work email, company) are defined in the `FIELDS` array in the script. In AEM these should come from the referenced form, not from this component's dialog.

## Porting to AEM

- **Render content on the server.** HTL should print every question, option, takeaway and explanation into the page markup so the content is indexable and readable without JavaScript. The clientlib then adds the pick-and-reveal behaviour. The prototype builds its markup in JavaScript only because it has no server.
- **Component structure.** One component with a dialog (fields above), an HTL template, a Sling Model, and a clientlib holding the CSS and JavaScript from `index.html`.
- **Scope the styles.** The CSS variables sit on `:root` and a few rules target bare elements (`body`, `h1`–`h3`, `p`, `ol`, `button`). Move the variables and those resets under the component's root class (`.tq`) so they do not affect the rest of the page.
- **Page chrome to drop.** The `.page` wrapper and the `.proto-note` line exist only for the prototype.
- **Several instances on a page.** The script uses fixed element ids (`tq-card`, `tq-list`, `tq-question`, `tq-first` and so on) and `document.querySelector(".tq")`. Make these per-instance if the component can appear more than once.

## Lead form and email delivery

- The prototype form sends nothing. In AEM it should post to Marketo (or whichever lead system is used). The place to add this is marked with a comment in the form's `submit` handler.
- Sending the form should trigger an email that carries the link to the executive summary page. The link is not shown on the quiz page, so the address is verified and the click ties later activity to the lead record.
- "Send it again" should re-trigger that email, with a limit on how often. "Use a different email" returns to the form.
- Keep the summary page out of search results (no-index, not in the sitemap), or people can find it without the form.
- Known limits to plan for: some emails land in spam; a forwarded email's clicks are attributed to the original recipient; corporate email scanners can register false clicks.
- Required fields: first name, last name, work email, company. Validation runs on submit; errors show under each field and focus moves to the first invalid field.
- The opt-in checkbox is optional and unticked by default. Check the consent and privacy wording with compliance for each market.

## Analytics

Each step pushes an event to `window.adobeDataLayer`. No personal details go into the data layer.

| Event | When | Payload (`takeawayQuiz`) |
| --- | --- | --- |
| `takeawayQuiz:answer` | An option is picked | `question` (1-based), `option` (letter), `correct` (boolean) |
| `takeawayQuiz:complete` | The last open question is answered | `score`, `total` |
| `takeawayQuiz:unlock` | The form passes validation | `optIn` (boolean) |
| `takeawayQuiz:resend` | "Send it again" is clicked | empty |

## Theming

The look is light only: brown text on white, with one copper accent. All colours and both font stacks are CSS variables at the top of the stylesheet.

| Variable group | Used for |
| --- | --- |
| `--bg`, `--ink`, `--muted`, `--line`, `--surface`, `--accent` | Page around the component |
| `--stage-ink`, `--stage-muted`, `--stage-line`, `--stage-tint`, `--brass` | Quiz text, rules, hover tint and accent |
| `--panel` | Highlight for the current row in the "Your takeaways" panel |
| `--good`, `--bad` | Right and wrong answer markers |
| `--paper`, `--paper-ink`, `--paper-muted`, `--paper-line`, `--paper-brass`, `--paper-bad` | Reveal sheet and form sheet |
| `--cover`, `--cover-ink`, `--cover-accent` | Report cover graphic |
| `--font-display`, `--font-body` | Headings and body text |

Typefaces load from Google Fonts: Cormorant Garamond (headings and numerals) and Hanken Grotesk (body). Replace the `<link>` and the two font variables to use brand fonts.

## Layout and breakpoints

| Width | Behaviour |
| --- | --- |
| Above 1040px | Quiz in two columns. Result takeaways in one row of five. Gate text beside the form. |
| 1040px and below | Result takeaways in rows of three and two. Gate text above the form. |
| 820px and below | Quiz stacks: heading, question, contents list. |
| 640px and below | Result takeaways stack in one column. |
| 520px and below | Report cover above the gate text. Form fields in one column. |

The result columns use CSS subgrid to align headings and text across columns. Browsers without subgrid still show the columns, without that alignment.

## Accessibility

- Answer rows, contents rows and all actions are native buttons or links, reachable by keyboard, with a visible focus outline.
- After an answer, focus moves to the reveal sheet, which has `role="status"`. After navigation, focus moves to the question or result heading.
- Right and wrong are shown with text labels and symbols as well as colour.
- Locked takeaways say so in text ("Takeaway 3", "Locked"); the decorative numerals and icons are hidden from assistive technology.
- Form fields have labels, `autocomplete` values, `aria-invalid` and linked error messages.
- Animations are switched off when the visitor has reduced motion turned on.

## Open questions

1. Which form fields are needed, and which system receives them?
2. Where will the executive summary page live, and which Marketo program sends the email?
3. Is the target AEM Sites (HTL) or Edge Delivery Services?
4. Brand colours and typefaces.
5. Final copy for the five questions, drawn from the outlook.
