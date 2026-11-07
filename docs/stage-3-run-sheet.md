# Stage 3 run sheet

The running order for a real multiplayer session from the [playtest plan](playtest-plan.md). Plan for 2–4 other people plus you, in several short sessions of about 15–30 minutes. At least some should not have been involved in designing the combat.

## Before the first session: let testers in

None of this has been tried yet. The steps come from Roblox's docs, last updated in June 2026, so if your screen differs, go by your screen.

1. In the Creator Dashboard, open the experience and go to **Configure > Settings > Audience**. Select **Limited > Playtesters**, but don't save yet. Roblox's docs list an age check, an account in good standing that's at least 2 days old, and the content maturity and compliance questionnaire.
2. If it asks for ID verification, 2FA or the 1,000 Robux fee, stop and find out why. The docs list those for reaching all ages, and don't say which requirements Playtesters needs.
3. In Studio, click **Collaborate**, search for each tester, set them to **Play** and save. They don't need to be your friend.
4. Save the audience setting.
5. Before the session, have one tester try the link while you're in the server. Check for their spawn line in the console.

Why this setting: playtest permission alone doesn't get anyone into a Private experience, and Limited > Friends would admit all your friends. The docs describe the lower requirement tier as reaching "16+ and Trusted Friends", so a younger tester might be blocked. Check that with the first tester.

## How testers join

- Give each tester their Play permission before the session, then send them https://www.roblox.com/games/134091394433436. The place isn't listed, so the link is the way in.
- You join first and stay in until everyone else has left.
- Check that Version History marks the version you mean to test as Published.

## What to watch for

The strongest signal is whether people want to play again. **Don't ask "was that fun?" until you've seen what they want to do.** After a fight or session, note whether anyone:

- asks for another round, or to swap sides or Kits
- tries the combo again, or goes after whoever just killed them
- experiments with Dash timing

For the Wet Reaction, note whether players:

- discover and use it naturally
- trigger it repeatedly during the session
- deliberately set it up once they understand it
- recognize what happened when it hits them

Rough goals, not pass or fail:

- fights of about 6–9 seconds
- an attack hit rate of about 35–60%
- players sometimes disengaging successfully
- Evade mattering without constantly triggering
- few attacks that look broken or unfair

Stage 2 left one thing for you to judge here: whether cones feel fair under latency. Also watch for spawn killing, since the spawn rule only changes if this stage shows it's still a problem.

The melee Basic Attack adds more to judge:

- whether swings near the edge of range feel unfair under real latency
- whether fights exceed the 6–9 second target, or end because neither player can reconnect
- whether players disengage too easily

## Comments to record

Write these down in the tester's own words, with who said it and in which session:

- "I definitely dodged that."
- "Why didn't that hit?"
- "I don't know what killed me."
- "I couldn't do anything."
- "That Dash felt really good."
- "I was trying to make them Wet so I could Arc Burst."

## Capturing logs

Read the lines starting `[Riftfall][Session]` in **F9 → Log → Server**. The console can't be copied from, so take screenshots.

1. Screenshot the spawn, kill, non-combat death and fight lines as they build up, and again before anyone leaves.
2. When a tester leaves, screenshot the JSON summary that prints for them.
3. **Stay in until everyone else has left.** The console is gone once the last player leaves.
4. Take a last screenshot after the final tester's summary, then leave.

Your own JSON summary only prints when you leave, so you probably won't be able to read it. Your fights still appear in the per-fight lines. No JSON summary has been read from a real server yet, so check that the first one looks right.
