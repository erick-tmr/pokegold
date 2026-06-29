# Timeless (Gold/Silver)

This fork removes the dependency on the cartridge's real-time clock (RTC) so that
in-game time is **frozen**: it never advances on its own and only ever changes when the
player sets it. Daily freebie events are made always-available, and the clock-setting
screen can be re-opened mid-game from the Pokégear.

These are the Gold/Silver equivalents of the changes from
[Pokemon_Crystal_Legacy_Timeless](https://github.com/erick-tmr/Pokemon_Crystal_Legacy_Timeless).

## 1. Frozen clock (RTC removed)

Gold/Silver normally read the MBC3 hardware RTC every time the clock updates. On
RTC-less hardware those reads/writes actually select an SRAM bank and corrupt save data,
so the cartridge type is changed to a plain `MBC5+RAM+BATTERY` and the clock primitives
are neutralized:

- `GetClock` (`home/time.asm`) returns a **frozen zero clock**.
- `LatchClock` / `SetClock` (`home/time.asm`) are no-ops.
- `StartRTC` / `StopRTC` / `SaveRTC` (`engine/rtc/rtc.asm`) drop their RTC register pokes.
- `Makefile`: cartridge header `MBC3+TIMER+RAM+BATTERY` → `MBC5+RAM+BATTERY`.

With a zero clock, `FixTime` makes the displayed time exactly equal the `wStart*`
offsets the player set in the clock UI, so the time/day are whatever you last set them to
and never drift.

Because the clock is frozen, the Pokégear **incoming-call** countdown is driven by the
play-time counter (`wGameTime*`) instead of the clock (`engine/overworld/time.asm`), so
scheduled calls (e.g. trainer rematches) still arrive while you play.

## 2. Always-available freebies

Daily events normally set a "done today" flag and only reset when the day advances.
Since the day never advances on its own, those events would otherwise be once-ever. The
per-day flag is removed from each so they stay repeatable. **Weekday and hour gates are
kept** — set the day/time you need with the Pokégear shortcut (section 3).

| Event | When it's available | Notes |
|-------|---------------------|-------|
| Fruit/berry trees | always | every tree refills on each check |
| Goldenrod underground bargain shop | per its schedule | repeatable while open |
| Haircut brothers (Goldenrod underground) | by weekday | weekday picks which brother |
| Indigo Plateau rival rematch | Mon/Wed | re-triggers on its tile — see the gotcha below |
| Trainer House battle | always | repeatable |
| Daisy's grooming (Pallet Town) | 3 PM | set the hour to 15:00 |
| Goldenrod Dept. Store rooftop TM (Return/Frustration) | Sunday | repeatable on Sundays |
| Bug Catching Contest | Tue/Thu/Sat | per-day entry cap removed; weekday kept |

Intentionally **unchanged**: the day-of-week sibling gifts (one-time-per-week *gifts*,
not repeatable freebies), the weekly Lucky Number radio show, the Mystery Gift daily
limit, and the Shuckle exchange (a time-advance trick, not a daily freebie).

**Bug Contest held prizes:** if you win with a full bag, the National Park gate officer
holds the prize and gives it back when you talk to them again. Vanilla gated that return
on the same "concluded today" flag we removed, so it's been re-pointed at the held-prize
events directly — the prize is returned by whichever gate officer is on duty (contest day
or not), and the contest stays repeatable.

**Gotcha — Indigo Plateau rival rematch:** on the rematch days (Monday/Wednesday) the
rival battle re-triggers *every time you step on its trigger tile* in the Indigo Plateau
Pokémon Center, because the once-per-day lock that normally stopped it is gone. That's
handy if you want to grind the rematch, but it means you can't slip into that Center to
heal — or walk through toward the Elite Four — without re-fighting him. If you want to heal
or head into the League uninterrupted, switch to any other day (not Mon/Wed) with the
Pokégear clock first; otherwise just beat him again to proceed.

## 3. Setting / changing the clock

There are three ways to open the clock-setting screen. All three write the same `wStart*`
offsets, so the choice is purely about convenience:

- **New Game** — the normal "set the time" prompt at the start of the game.
- **Title-screen clock reset** — the vanilla "reset the clock" chord on the title screen.
- **Pokégear shortcut** *(Timeless addition)* — open the Pokégear, land on the **Clock**
  card, then **hold SELECT and press UP**. The same clock-setting screen opens without
  having to return to the title — useful for mid-route adjustments (flip to night for a
  Hoothoot, swap to Sunday for the bargain shop, advance the day, etc.). The combo is
  deliberately obscure so the time can't be changed by mistake, and the screen still asks
  for YES/NO confirmation before applying.
