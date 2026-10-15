# Fair Share: Design Specification

## Problem

University students working in project teams often end up with very unequal contributions. People put in less effort in a group when nobody can see what they personally did (Latané, Williams & Harkins, 1979). In student teams, commitments are usually made out loud, lost in a group chat, or left vague, so missed work only comes to light days or hours before the deadline. Peer evaluations arrive after the project is over, and professional tools ask for setup and upkeep that students rarely keep going.

In informal interviews, three classmates described the same pattern: a teammate's missing or mismatched work was discovered only days or hours before the deadline, and the team had to redistribute tasks or reorganize at the last minute. Student teams do not have a light, friendly way to see who has committed to what and whether it is getting done. (See README.md for the full problem statement and these findings.)

## Solution

Fair Share is a small web app where a team posts weekly commitments, each owned by exactly one member, and everyone can see their status. It is deliberately simple. There are no accounts, no passwords, and no notifications. It works on phones and computers with one responsive layout.

### Concepts

* **Team:** a named space with a join code of 6 characters. The code uses the letters A to Z and the digits 2 to 9 (no 0, 1, O, or I, since they look alike).
* **Member:** a person in a team, identified by a display name of 1 to 20 characters.
* **Commitment:** a task with a title (1 to 80 characters), a due date, one owner (a member), and a status of either Open or Done.
* **Overdue:** an Open commitment whose due date is before today, using the viewer's local time.

### Identity

There are no accounts. When someone creates or joins a team, they pick a display name, and the browser remembers "you are *Name* in team *CODE*" using local storage. Opening the app again in the same browser goes straight to that team's Board. The "Leave team" button on the Board clears this memory on that device. It does not delete the member or their commitments.

### Screen 1: Home

```
+===============================+
|          Fair Share           |
|    Make group work visible.   |
|                               |
|     [ Create a team ]         |
|     [ Join a team   ]         |
+===============================+
```

* Shows the app name, the tagline "Make group work visible.", and two buttons.
* "Create a team" opens Screen 2. "Join a team" opens Screen 3.
* If the browser already remembers a team, the app skips Home and shows Screen 4 (the Board).

### Screen 2: Create team

```
+===============================+
| < Back                        |
| Create a team                 |
| Team name  [______________]   |
| Your name  [______________]   |
|              [ Create ]       |
+===============================+
```

* Team name is 1 to 40 characters. Your name is 1 to 20 characters. Spaces at the start and end are trimmed.
* "Create" stays disabled while either field is empty after trimming.
* On success, a new team and its first member are created and a join code is generated. The app then shows Screen 4 with a banner that appears once: "Team created! Share this code with your teammates: ABC234", along with a "Copy code" button. The code always stays visible in the Board header afterward.
* Errors appear in red text under the Create button:
  * If a field hits its length limit, it stops accepting characters and shows "Max 40 characters" or "Max 20 characters".
  * If the server or network fails: "Couldn't create your team. Check your connection and try again." The fields keep what the person typed.
* "Back" returns to Home without saving anything.

### Screen 3: Join team

```
+===============================+
| < Back                        |
| Join a team                   |
| Team code  [______]           |
| Your name  [______________]   |
|               [ Join ]        |
+===============================+
```

* Team code is 6 characters and is changed to uppercase automatically. Anything that is not a letter or digit is ignored as the person types. Your name is 1 to 20 characters.
* "Join" stays disabled until the code has 6 characters and the name is not empty.
* On success, the person becomes a member and sees Screen 4.
* Errors appear in red text under the button:
  * Code not found: "No team with that code. Check the code and try again."
  * Name already taken in that team (ignoring upper and lower case): "Someone in this team already uses that name. Try another."
  * Team is full (the limit is 8 members): "This team is full (8 members)."
  * Network failure: "Couldn't join. Check your connection and try again."

### Screen 4: Board (main screen)

```
+=====================================+
| Team Name            Code: ABC234 c |
| You: Sam                [Leave team]|
|=====================================|
| [ + Add commitment ]                |
|                                     |
| OVERDUE                             |
| [ ] Draft intro slide     Jo  Oct 12|
|                                     |
| OPEN                                |
| [ ] Write user survey     Sam Oct 18|
| [ ] Set up repo           Alex Oct 20|
|                                     |
| DONE                                |
| [x] Pick team name        Sam Oct 10|
|                                     |
| [ Team summary ]                    |
+=====================================+
```

* **Header:** the team name, the join code with a copy button (it shows "Copied" for 2 seconds after tapping), "You: *Name*", and a "Leave team" button.
* **Sections:** commitments are grouped into Overdue (red heading), Open, and Done, in that order. A section with nothing in it is hidden. Overdue and Open items are sorted by due date, earliest first, and ties go to whichever was created first. Done items are sorted with the most recently completed first.
* **Rows:** each row shows a checkbox, the title, the owner's name, and the due date written like "Oct 18". The year only appears if it is not the current year.
* **Checkbox:** only the owner of a commitment can use it. Tapping it on someone else's row does nothing except show "Only Jo can mark this done." for 3 seconds below the header. Checking an Open item moves it to Done and records the completion time. Unchecking a Done item moves it back to Open.
* **Editing:** tapping the title of a row opens Screen 5 if the viewer is the owner. For anyone else, nothing happens.
* **Refreshing:** the Board reloads its data every 30 seconds and whenever the browser tab comes back into focus, so teammates' changes show up without a manual reload.
* **Empty state:** when there are no commitments at all, the lists are replaced by "No commitments yet. Add the first one!" under the Add button.
* **Loading:** while data loads, the lists are replaced by "Loading…". If loading fails, the screen shows "Couldn't load your team." with a Retry button that tries again. If the team no longer exists, it shows "This team no longer exists." with a "Back to home" button that clears the remembered team.
* **Leave team:** asks "Leave this team on this device? You can rejoin with the code." with Cancel and Leave buttons. Leave goes back to Home.
* **Team summary:** opens Screen 6.

### Screen 5: Add or edit commitment

```
+===============================+
| Add commitment                |
| What will you do?             |
| [__________________________]  |
|                       0/80    |
| Due date  [ Oct 18 ]          |
| [Cancel]            [Save]    |
|           (Delete)            |
+===============================+
```

* Shown as a pop up over the Board. The owner is always the current user. People can only make commitments for themselves, on purpose, so that commitments stay voluntary.
* **Title:** 1 to 80 characters, trimmed, with a live counter such as "12/80".
* **Due date:** uses the browser's date picker and defaults to 7 days from today. When adding, a date in the past shows "Due date can't be in the past." When editing a commitment that is already overdue, its existing past date is allowed.
* **Save:** stays disabled while the title is empty. On save, the pop up closes and the Board updates.
* **Delete:** only appears when editing. It asks "Delete this commitment?" with Cancel and Delete buttons. Deleting removes it for everyone.
* **Errors:** if the network fails, the pop up shows "Couldn't save. Try again." and keeps everything the person typed.
* **Cancel:** closing the pop up with Cancel or by tapping outside it discards the changes. If there are unsaved changes, it first asks "Discard changes?" with Keep editing and Discard buttons.

### Screen 6: Team summary

```
+===============================+
| < Back to board               |
| Team summary                  |
|                               |
| Alex  (you)  ##########  3 of 3|
| Jo           ##........  2 of 5|
| Sam          ####......  4 of 6|
|                               |
| Overdue: 1                    |
+===============================+
```

* There is one row per member, with a progress bar and "X of Y", where Y is the number of that member's commitments (Open plus Done) and X is how many are Done. A member with no commitments shows "No commitments yet" and no bar.
* Rows are in alphabetical order by name. The bar is one neutral color, never red or green, and the current user's row is marked "(you)".
* At the bottom, "Overdue: N" shows the team's total number of overdue commitments. It is hidden when N is 0.
* The summary shows counts only. It never ranks members or labels anyone, because the goal is to make work visible and not to shame people.

### Content and visual style

* **Font:** the device's default font. Body text is 16px and headings are 20px bold.
* **Colors:** white background, near black text (#1a1a1a), blue primary buttons (#2563eb) with white text, red for the Overdue heading and error text (#b91c1c), and gray for disabled controls (#9ca3af).
* **Layout:** a single column, 480px wide at most, centered on the screen. Every tappable control is at least 44 by 44 pixels.
* **Language and input:** all text is in English. There is no sound. On a keyboard, Enter submits forms and Esc closes pop ups.

### Out of scope for the first version

Accounts and passwords, notifications or email, comments, file attachments, giving tasks to other people, more than one team per device, instructor views, changing the team name, and removing other members.

## References

* Latané, B., Williams, K., & Harkins, S. (1979). Many hands make light the work: The causes and consequences of social loafing. *Journal of Personality and Social Psychology, 37*(6), pp. 822 to 832.
