# Notes

What I found while playing Snake, and what I asked the agent to change.

## 1. The board does not fit in the window

**Playing:** with the browser zoomed in, the page gets scrollbars and the layout
scrolls while I play. Part of the board ends up off screen and I never see the
score during a game.

**Asked the agent:** make the game fit the window — board scaled to the space
available, centred, page never scrolling, cells kept square.

**First attempt:** it rewrote the layout and explained what it had done. It tried
to verify in a headless browser, found no Node available, and could not check
anything itself; it opened the file and asked me to confirm. I hard-reloaded and
nothing had changed.

**Second attempt:** I told it plainly that nothing had changed and to check what
the page actually does instead of describing what the code should do. This time
it found Chrome on my machine and drove it headless at 100%, 150% and 200% zoom.
No overflow in Chrome. It then found a real flaw by re-reading its own CSS: the
fix only worked if a JavaScript resize event fired, with no CSS fallback. It
added `overflow: hidden` and size limits, swapped the resize listener for a
`ResizeObserver`, and proved the clamp by forcing the canvas to 900px inside a
240px box.

**Result:** still not fixed for me — see section 2, which turned out to be the
real cause.

## 2. The arrow keys were scrolling the page

**Playing:** testing it again gave me the detail that mattered: the scrollbar
moves on every left or right arrow press. The page was not overflowing because
of the board — the arrow keys were steering the snake and scrolling the browser
at the same time.

**Asked the agent:** told it exactly that, and to stop the game's key handling
from letting the browser scroll.

**Result:** fixed for the arrow keys. The snake steers correctly and the document
no longer scrolls. But the view still slid sideways while I was zoomed, which led
to section 3.

## 3. What was left could not be fixed in the page

**Playing:** even after the keyboard fix, the view still panned to the left while
I was zoomed in, and losing appeared to reset my zoom.

**Asked the agent:** why the page still has a horizontal scrollbar and what is
moving the view now that it is not the arrow keys.

**What happened:** it simulated a whole play-through in headless Chrome, holding
arrow keys down and sampling the scroll position continuously. Every key press
showed `defaultPrevented: true` and the scroll position never moved. It could not
reproduce my problem — and instead of guessing, it asked me one question: was I
zooming with the keyboard or with a trackpad pinch? With the pinch, the answer
fell out. Pinch zoom does not reflow the page; it magnifies what is already
drawn, and panning inside that magnified view is handled by the browser and the
operating system, outside the page entirely. No JavaScript can intercept it. So
that part is not a bug in the game and has no fix in the code.

It did find the real cause of the second symptom: the Restart button went from
taking no layout space to taking real space the moment I lost, so the board
genuinely shrank at that instant — which looks exactly like the zoom resetting.
It measured this live (canvas width dropping from 340 to 300 while the browser's
own zoom level stayed unchanged) and fixed it by reserving the button's space at
all times.

**Result:** fixed. The board keeps the same size before, during and after game
over, so losing no longer looks like the zoom resetting.

## 4. The real reason I was zooming

**Playing:** once the zoom question was settled, I noticed why I kept zooming in
the first place: the board looked small at normal size. And when I zoomed, the
top of the board fell outside the view and I died because I could not see where
the snake was going.

**Asked the agent:** stop treating this as a zoom problem — make the board use as
much of the window as it can at normal size, staying square, so that zooming is
never necessary to play comfortably.

**Result:** fixed at normal size. It merged the title and score into one row and
trimmed the margins to reclaim vertical space; the board went from about two
thirds of the window height to roughly 80%. I can now see the whole game without
zooming. Zoomed in it still does not fit — but by then we knew why. The fix was
to remove the reason I was zooming, not to fight the zoom.

## What playing taught me about working this way

Four rounds went into what looked like one symptom, and the pattern was the same
each time: the agent could reason about the code far faster than I could, but it
could not press a key and watch what happened. Every time it got stuck, what
unblocked it was an observation only available from playing — first that the page
moved *on arrow keys*, then that I was zooming *with a pinch*, then that I was
zooming at all because the board was too small.

The first round is the one I would flag to anyone starting this. The agent said
"this looks correct", and that was a statement about the code it had just
written, not about the page. It had no way to see the result. It only became
useful once it went and found a browser it could actually drive — and even then
it was testing in Chrome while I played in Safari, so its evidence and my
experience still were not the same thing.

Two moments were worth more than any of the fixes. One was when it stopped trying
to fix something and told me it could not be fixed from inside the page, and why.
The other was when it asked me a question instead of guessing — keyboard zoom or
pinch? — because that single answer explained three rounds of failed attempts.

## Known, not fixed

At normal size the game fits and plays correctly. If you zoom in with a trackpad
pinch, part of the board goes out of view and cannot be brought back by the page:
pinch zoom magnifies what is already drawn, and the panning is handled by the
browser and the operating system, outside any JavaScript the page can run. I am
leaving it as it is, on purpose — the board is now large enough at normal size
that zooming is not needed.