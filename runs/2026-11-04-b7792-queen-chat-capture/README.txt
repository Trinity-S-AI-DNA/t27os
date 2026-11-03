Queen chat capture -- T27 OS build 7792 (2026-11-04)
worktree: ~/t27/.claude/worktrees/lead-queen-reconcile (branch lead/queen-reconcile)
build dir: ~/.t27os/stable-src/b7792 (T27OSv.app, CFBundleVersion 7792, ad hoc signed)

Files
  snapN-7792-normal.png   the QUEEN CHAT dock, echo OFF: two receiver cards restored from
                          ~/.t27os/queen/receiver.t27q (the file was written through the
                          receiver's own qr_accept + qr_seal by a scratch .t27 program, then
                          read back by the app at start). Composer says "Queen panel not
                          available"; nothing was written to any pane.
  snapN-7792-normal.txt   the same frame's element tree (t27os-ui-snapshot, build 7792)
  snapN-trace.log         observatory trace: env_present -> ... -> capture_completed
  snapF-7792-fixture.png  echo test aid ON: the labelled fixture conversation; every card
                          carries "test fixture" ("Queen - test fixture, not the Queen").
  snapF-7792-fixture.txt  and snapF-trace.log -- the same for the fixture run
  baseline-glass_window.log    t27b test specs/t27os/glass_window.t27 on the branch base:
                               112 passed, 3 failed (the three stale region asserts)
  after-glass_window.log       after the reconciliation: 115 passed, 0 failed
  after2-glass_window.log      after the echo-fixture test aid: 115 passed, 0 failed
  after-queen_receiver.log     queen_receiver 20/20
  after-pane_slots.log         pane_slots 8/8
  after-chat_surface.log       chat_surface 17/17
  after-chat_identity.log      chat_identity 12/12
  after-queen_verdict.log      queen_verdict 17/17
  after-queen_chats.log        queen_chats 28/28
  after-shell_dock.log         shell_dock 23/23

How the frames were taken (no screencapture, no screen permission)
  env T27OS_OBSERVATORY_ROOT=<dir> T27OS_BUILD=7792 T27OS_QUEEN_OPEN=1 [T27OS_QUEEN_ECHO=1] \
      ~/.t27os/stable-src/b7792/T27OSv.app/Contents/MacOS/T27OS &
  touch <dir>/want   # one file asks for one frame; the app writes snap.png + snap.txt itself

Scope, honestly
  window -> receiver: the READ/RESTORE direction is demonstrated in snapN; the submit (WRITE)
  path is wired in the same table and card code (qr_accept -> qr_card_text -> qr_seal -> file)
  and covered by tests, but a live typed submit was not performed (no input automation).
  receiver -> executor: BLOCKED -- no designated executor; 16/16 panes busy; the candidate
  pane 9 (session 0bb2efcf) was re-validated (state 128, flags 32, not live) and NOT closed;
  the owner decides.
  result -> verdict -> chat: BLOCKED on the same executor; the verdict machine and its card
  labels are in place (queen_verdict 17/17).
  GitHub sign-in: the "TEMPORARILY COMMENTED OUT" timing lived only in an abandoned working
  copy; this branch carries the working code (no TEMPORARILY line anywhere). No avatar yet:
  the device-flow authorization is a human step and was not performed.
  Echo: a test aid only, off by default; the fixture run is labelled on every card.

Compliance on the owner's machine: no owner pane was closed; the shared PTY server (PID
99422) was not restarted; no stable slot was changed (the build lives in stable-src/b7792);
the seeded receiver file was removed after the capture, so the owner's next start is clean.
