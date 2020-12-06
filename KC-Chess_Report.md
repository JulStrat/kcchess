# KC Chess

by R. Kevin Phillips and Craig S. Bruce

for Dr. J. D. Horton / CS 4993

April 6, 1990

## PREFACE

It was our, R. Kevin Phillips and Craig S. Bruce's, intention to design
and implement a working computer chess game.  Contained in this paper are
the methods used, alternatives discarded, and discussions of problems that
we encountered while writing the program.

## OUTLINE

```
1. Introduction

2. Data Structures
   2.1. Board Data Structure
   2.2. Possible Moves Data Structure
   2.3. Move Data Structure
   2.4. Player Data Structure
   2.5. Game Data Structure

3. Human Movement
   3.1. Creating the MoveList
   3.2. Disallowing Certain Moves

4. Computer Movement
   4.1. Zero-Ply Lookahead
   4.2. One-Ply Lookahead
   4.3. Two-Ply Lookahead
   4.4. Generalized N-Ply Lookahead
   4.5. Search Tree Pruning
   4.6. Finishing Touches

5. Program Features
   5.1. Set Up Board
   5.2. TakeBack and UnTakeBack
   5.3. Player Settings
   5.4. Options
   5.5. Watch Game
   5.6. Go To Move Number
   5.7. File System
   5.8. Hint
   5.9. On-Line Help
   5.10. High Resolution Graphics

6. Program Limitations
   6.1. Horizon Effect
   6.2. Scoring Deficiency

7. Conclusion
```

## 1. INTRODUCTION

It is very difficult for two people to be able to merge their ideas into
a single working program as large as this one.  It is for this reason that
Craig acted as the leader of the programming portion.  He, with my input,
made the large-scale decisions of how the program would be modularized.
From there the smaller aspects of the program could be assigned and as long
they interfaced properly, the program would work.  It was decided that
because Craig's larger contribution during the programming of the project
that I prepare most of the report as well as present the project at the
seminar.

When I first thought about trying to write a program to play chess I
could not even think where to start.  It seemed an overwhelming problem and
there were so many places to make errors that could take hours and hours to
repair.  After mulling over the design for some time, we began to program.
When we were finished, we were surprised that we encountered so few
problems.  Turbo Pascal was chosen for it's powerful built-in libraries,
it's interactive nature and it's impressive speed.

## 2. DATA STRUCTURES

Pascal, being very rich in data structures, gave an almost unlimited
choice for representing a chess game.  The problem was trying to choose the
structures that would allow the best combination of flexibility, economy and
speed.

### 2.1. BOARD DATA STRUCTURE

The main decision, in terms of data structures, was how to represent the
actual chess board.  The first structure to come to mind was simply an 8x8
array with each element corresponding to a square on the board.  As obvious
as this was, it was not the structure that was used.  We chose instead, for
later ease of determining illegal moves, that the array be 12x12 (-1..10,
-1..10), where the board occupies the center 8x8 squares. With the 8x8 array
it is necessary to determine if the move will go off the board before the
move is made, so as to not access a nonexistant array element.  The 12x12
board allows the moves to be made and then see that the piece has moved to
an illegal square.  The latter method allows us to make only a single test
of legallity, whether the square is legal or not. With the former, both the
row and column indices are required to be between 1 and 8.

Upon doing some research on the subject, another strain of methods was
discovered.  These methods use a one dimensional array.  The major benefit
of these methods was in determining where a piece can move to. For example,
rooks can move: +1, -1, +12, -12.  So instead of having to store both the
row and column displacements, as is needed in the array implementation, only
one value is needed for describing the move.  Also, since only one subscript
needs to be resolved in accessing a square on the board, the 1-D array would
work faster.  However, we chose the 2-D matrix because it is more natural
and easier to follow.

The next decision that had to be made was: what is required in each board
element.  Each board element must describe the type of piece in the square
and its color.  An enumerated type called PieceImageType is used, which
allows the program to be most readable.  A type, PieceColorType, is declared
to contain either the value white or black.  As stated earlier, to determine
an illegal move, there is a ValidSquare variable in the structure.  A
variable to determine whether the piece has moved or not is also included.
It is needed for pawns trying to move two squares and for castling.

### 2.2. POSSIBLE MOVES DATA STRUCTURE

Trying to find the possible moves available to a given piece could be a
very difficult task if the proper data structure is not chosen.  An array,
indexed by PieceImageType, is used.  Each element in the array consists of
the range of the piece, the number of directions and the direction vectors
themselves.  The range is not simply the number of squares that the piece
travels, rather it is the number of times that the direction vector may be
applied.  To illustrate, the queen has a range of 7, not including her
present square.  She can move in eight different directions, right, left,
up, down, and along the four diagonals.  The direction vectors are simply
the row and column displacement, from the starting square for the smallest
possible increment in that direction. The direction vectors for all pieces
are listed below.  Note that pawns are not listed here since all their moves
are special.

```
PIECE   RANGE    VECTORS
------  -----    -------
King      1      (-1,-1) (-1,0) (-1,1) (0,-1) (0,1) (1,-1) (1,0) (1,1)
Queen     7      (-1,-1) (-1,0) (-1,1) (0,-1) (0,1) (1,-1) (1,0) (1,1)
Rook      7      (-1,0) (1,0) (0,-1) (0,1)
Bishop    7      (-1,-1) (-1,1) (1,-1) (1,1)
Knight    1      (1,-2) (2,-1) (2,1) (1,2) (-1,2) (-2,1) (-2,-1) (-1,-2)
```

### 2.3. MOVE DATA STRUCTURE

Now that we have the board and the pieces described, we must be able to
describe any move generically.  Again Pascal's powerful data structures came
to our aid.  A MoveType is declared to consist of the row and column that
the piece was moved from and the row and column that the piece was moved to.
These are needed for obvious reasons.  The type of the piece moved as well
as the type of the piece taken are also stored in this structure.  These
values are required for the Takeback/Untakeback feature since when a move is
taken back, the piece that was taken must materialize from nowhere to its
original square.  The piece that was moved seems to be redundant but
consider pawn promotion.  When the move that promoted a pawn is taken back,
the queen (or whatever the pawn was promoted to) must disappear and the pawn
must again come from nowhere.

### 2.4. PLAYER DATA STRUCTURE

A PlayerType is also declared to contain all the pertinent information.
The player's name, color, skill level, last move, check status (ie. OK or IN
CHECK) and elapsed time are all stored in this structure and are printed to
the screen for the players' viewing.  It is also necessary to store the type
of player (ie. Human or Computer), the current position of the players king
(done for simplicity and speed of determining check) and the position of the
player's cursor.  The cursor position is used to keep the cursor where the
player's last piece was moved.  A flag also exists for each player for the
consideration of positional evaluation.  Note that the skill level and the
positional evaluation flag are used for human players to give the computer
the basis for finding the human player a move (the Hint command) if he
should ask.

### 2.5. GAME DATA STRUCTURE

The last structure is the structure that is written to disk when a game,
complete or incomplete, is saved.  Because the GameType structure must be
able to reconstruct every aspect of a game, it is by far the largest of any
of the structures.  It contains the descriptions of the players (PlayerType)
as described earlier, and it contains the list of all the moves and how many
moves were made during the game.  There is also an array of booleans that
tell if the player to move is in check in the corresponding move.  Also,
GameFinished, TimeOutWhite/Black, Stalemate, NoStorage tell if the game is
complete and why.  NonDevMoveCount counts upwards for the fifty move
stalemate rule.  The final board is also saved so that the game can be
un-made all of the way back to the standard setup or to any move in the
entire game.

In addition to these values in the GameType structure, there are options
that can be set to what ever is desired.  There are flags here for allowing
or disallowing en passent, and for enabling or disabling the sound.  The
player of the game may choose to change the settings for the flash count,
watch delay and the time limit.  The flash count gives the number of times a
piece is flashed before and after it is moved.  The watch delay gives the
number of milliseconds to wait between displaying moves in Watch Mode.  The
time limit gives the maximum elapsed time allowed for either player.  A time
limit of zero means no limit is set.

## 3. HUMAN MOVEMENT

When it is time for a human player to make a move, the first thing to be
done is to calculate all possible moves.  That is all moves that abide by
the directional vectors and stay on the board.  There are other moves that
do not fall into a 'nice' catagory.  A few of these are castling, pawn
promotion, en passent and whenever a pawn captures another piece.

### 3.1. CREATING THE MOVE LIST

To make this list, each player's piece is examined individually and all
moves for that piece are created.  With the help of the PossibleMovesType
structure, it is a relatively simple process to go through all directional
vectors.  From the starting square, each direction vector is projected
outwards until the maximum range of the piece is reached, the piece goes off
the board or the piece runs into another piece.  If the other piece is a
piece of the opposite color then taking it can be counted as a possible
move.  This relatively small amount of code will handle most of the moves
made in a game but it can not be easily adapted to handle the less rigid
moves.

A pawn is only allowed to move diagonally forward when it captures an
enemy piece.  Because there must be a piece in the diagonal square, caputure
by pawns is not compatible with the more general move generator and must be
examined specially.  Even advancing a pawn one space forward is a special
case since if the pawn moves to the last row it must be promoted.  Advancing
the pawn two squares requires a special check to make sure that it has not
been moved before.  The move known as 'en passent' must also be examined
separately because it can only happen immediately after an enemy pawn
advances two squares.  When determining if this move is possible it is
necessary to look at the previous move of the enemy.  If it was advancing
the specific pawn twice that is to be captured, then the en passent move is
possible.

Castling is also a very special move since two different pieces have to
be moved to carry out castling, and it can only be performed when neither
the king nor the rook involved have yet moved.  This is relatively easy to
see as the HasMoved flag associated with each piece can be checked.
Castling is also further complicated since the squares between the king and
the rook must be empty, the king can not be in check before the move, after
the move and can not move though check.  All of these conditions must be
checked explicitly.

### 3.2. DISALLOWING CERTAIN MOVES

Not all of the moves generated above are actually legal.  All moves that
would result in the player's king being in check are not allowed. Thus,
after all of the possible moves have been generated, all of the illegal ones
must be eliminated.  This is done by making each move and determining if the
king has been put in check.  This is done by exhaustively examining all
squares that could possibly attack the square that the king is on.  Now all
legal moves for the player have been determined and are stored in the
MoveListType structure.  Although this is tedious, no better way was found.

The next step is very trival in comparision to generating the list of
legal moves.  The player's move is merely read in and tested to see if it is
indeed in the list of legal moves.  If it is not in the list then the player
is attempting an illegal move and he should be informed and forced to choose
another.  If the move is valid then the move can be made and the enemy gets
his turn to make a counter move.

## 4. COMPUTER MOVEMENT

Perhaps the most interesting feature of a chess program is its ability to
seemingly think.  Let it be stated that there is no magic involved, only
extensive computation.  A large tree structure is generated, traversed, and
valued in order to compute which move is the "best".  Presented here is the
evolution of the Computer Movement routine for our project.

### 4.1. ZERO-PLY LOOKAHEAD

In the beginning, when we created the computer movement thinking routine,
it was simple and stupid.  It would call for the generation of the move list
and would then pick a move randomly (a zero-ply lookahead). This allowed the
other higher level routines which called upon this routine for the move, to
be implemented.  Needless to say, the moves selected were completely
meaningless.  The pieces would wander aimlessly about the board and
occasionally do something useful.

### 4.2. ONE-PLY LOOKAHEAD

After testing the higher level routines, the thinking 'stub' was modified
to search the move list for the move that captured the enemy's highest
valued piece.  The values were based on the old standards PAWN = 1, KNIGHT =
3.5, BISHOP = 3.5, ROOK = 5, QUEEN = 9, and KING = infinity. This
facilitated a one-ply lookahead.  The result, of course, was that a piece
would take another piece whenever it could, with no concern for its own
safety (it would often walk into an ambush), and when no enemy piece could
be taken, the pieces would wander around the board aimlessly, as before.
Since the move list was always generated in the same order, to add some
variety to the game the move list was randomized (using a linear algorithm)
before searching.  Thus, tie scores would be chosen between randomly.

Also, the rules about checking and other special moves were not yet
implemented, so the game would continue after the enemy's king was taken,
until all of his pieces were taken.  No effort was made to figure out who
actually won, either.  With the one-ply lookahead, the games would finish
fairly quickly, because there was such a take-frenzy.

### 4.3. TWO-PLY LOOKAHEAD

At this time it was hypothesized that a two-ply lookahead would stop
important pieces from walking into ambushes.  The hypothesis was proven
correct when a hard-coded brute force routine was implemented.  This routine
had to be implemented differently from zero- and one-ply since it had to
examine all of the moves possible after the first move had been made.
Searching required two nested loops, where the first loop would go through
all of the first moves, remember the take-score, and then actually make the
move and update the internal board representation.  The inner loop would
then generate all of the moves possible for the opposite color to the first
move, and would search through this list and remember the score for taking
the highest valued piece.  After this inner search, the first move was then
taken back and the internal board updated, and the outer loop continued.

The score for the first move was the value of the piece taken in the
first move minus the value of the piece taken in the second move.  The
advantage over the one-ply lookahead was very obvious: the moving player
would stop walking into ambushes, since it would SEE that if it took a
lesser, protected piece, then the enemy would take the the first piece
right back.  In fact, this 'chickening out' would go a little too far,
since in two plys the player to move cannot see his own protection of the
square that he just took an enemy piece on; at the time it was thought
that this would require a three-ply lookahead.  In the event that no
pieces could be taken, the pieces would wander around aimlessly.  A human
with any skill at chess could easily prey upon these limitations and
defeat the computer.

Assuming an average of 35 moves for each level of this tree structure,
the two-ply lookahead would have to examine 35 * 35 = 1225 moves to find the
"best" one.  At the time, it took about 15 seconds to perform this search on
an IBM PS/2 Model 70 (a 25 MHz personal mainframe).

The special moves of castling, pawn promotion, and en passent were
implemented at this time, with relative ease, and with a relatively small
drain on thinking speed.  However, the rules concerning checking were much
more expensive to enforce, since the algorithm had to discard any moves in
the move list that would result in the player being in check after the move
was made.  The easiest way to do this was to actually make the move and
generate the enemy's countermove list.  If any of these moves were capturing
the first player's king, then the first move was invalid and would not be
considered.  This also had to be done for each inner-loop move, which
effectively meant that a three-ply search was generated, even though only
two plys could be used for evaluation.  The Model 70 thinking time increased
by three times to about 45 seconds.  However, at about this time an option
in the compiler was discovered that made the program run about three times
faster, which trimmed the thinking down time to 15 seconds again.

### 4.4. GENERALIZED N-PLY LOOKAHEAD

At this time, the focus of the project became improving the thinking time
and the quality of the moves.  A flexible, recursive thinking routine was
needed which could be used for a tree search of any number of plys. The
ingredients would be the same as before; the move list for the current
player would be generated, and in a loop, each move would be made and the
points for the move would be remembered.  Then, a recursive call would be
made to evaluate the enemy's counter move, and the total score of the first
move would somehow be calculated and the maximum score and move would be
remembered.  Then, the first move would be un-done, and the first move loop
would continue until all moves have been tried.  The maximum first level
move would then be returned by the computer thinking routine, and other
parts of the program would make and display the move for the user to see.

Since the consulted published articles did not score moves in quite the
same way (their method was not completely understood or believed), a scoring
method had to be figured out the hard way.  The first trial involved simply
adding the 'friendly' move scores and subtracting enemy move scores from a
global variable, at each ply.  The result, just as simply, was the best
score for the player to move, assuming that the enemy makes his WORST
countermoves.  This is exactly opposite to what is required for a true
minimax search: to maximize the score of the player to move, under the
assumption of BEST play by BOTH sides.

Trying to make the global total score come closest to zero was also an
invalid idea, since an important feature of the scoring method is that a
high score should be registered if the friendly player makes a 'killer' move
for which the enemy cannot compensate.  Under the zero-sum scoring method,
this killer move would be rejected in favor of a 'tamer' move in which
neither side gains advantage.

During a brain-storming session, the simple solution was discovered. Take
the score of the friendly player's MOVE, and subtract from that the score of
the enemy player's best counter-LINE of play, and find the move that gives
the maximum score.  The counter-LINE of play is defined as taking the score
of the enemy's counterMOVE and subtracting from that the score of the
initial friendly player's best counter-LINE to the countermove, and finding
the maximum score, and so on.  Note that this definition is recursive, and
that it satisfies the principal of best play by both sides.  The limit of
the recursion is the number of plys to look ahead or the end of the game,
whichever happens first (almost always the ply limit).  Note that if the
current line of play is being considered as an N-ply search, then the
enemy's best counter-line will be determined with an (N-1)-ply search.  See
the Appendix for an example of a search tree.  The following pseudo-code
returns the maximum score of the player to move.  The code for searching the
first ply of the lookahead should also return the move corresponding to the
maximum score.  This will be the move selected by the thinking algorithm.

```
Search (Player, Depth);
    If Depth = 0 Then
        return (0);
    Else
        BestScore := -infinity;
        Generate the move list for Player as per current board setup;
        For each legal move in the move list do
            Make the current move and get MoveScore;
            Score := MoveScore - Search (enemy of Player, Depth - 1);
            If Score > BestScore then BestScore := Score;
            UnMake the current move;
        End For;
        Return (BestScore);
    End If;
End Search;
```

With this algorithm implemented, the three-ply search was tested.  It
offered many advantages over a two-ply search.  It would see if it had the
necessary backup to take a protected enemy's piece, and it would even 'fork'
an enemy's piece if it could (e.g., moving a knight to an unprotected square
such that it attacks both the enemy's king and queen; the king must move and
knight takes queen).  The price for this new ply of vision was execution
time, of course.  The Model 70 time was typically between two and three
minutes.  The average number of moves examined was 35 * 35 * 35 = 42,875.  A
four-ply search was beyond the realm of human patience.

A new problem was noticed with the three ply search.  If the third move
of the current lookahead was a take, then it was assumed that the piece
could be taken without cost; no check was made to see if the piece taken was
protected.  Thus, a powerful piece would often make foolish moves because it
thought that it could take an enemy's piece for free in the powerful piece's
next turn; some pieces were even sacrificed for this purpose.  In reality,
when the next turn came around, the player to move saw that the piece it
thought it could take was actually protected, and often made the same
mistake in its new search.

To the rescue came a new routine which would tally the number of other
pieces protecting or attacking a certain piece.  Now, if the last move in a
search line of play was a take and the piece taken was protected, it was
assumed that the player to move would lose his piece if he took.  This new
attack and protect counter was also used to see if a king was in check (if
there was one or more attacking pieces).  It was a much more elegant
solution than checking the enemy's countermove list for taking a king, and
it made the computer think four times faster.  The three-ply Model 70 time
was down to 30 seconds.

### 4.5. SEARCH-TREE PRUNING

At this time, optimizing the code had mostly reached its limit.  The next
step was to minimize the problem.  It is well known in artificial
intelligence that large sections of a two player game tree can be cut off
without affecting the outcome of the search.  The idea is that if any of the
enemy's countermoves to any of the possible current moves is too beneficial
for the enemy, then the current player will not select that current move;
one more beneficial to the current player will be selected. Thus, the
enemy's countermove list needs only be searched until a move-line which
exceeds the current player's enemy score tolerence limit is found.  The rest
of the enemy's countermove list and the subtrees hanging off of them need
not be examined.  The searching algorithm augmented with the cutoff idea
follows:

```
CutoffSearch (Player, Depth, CutoffValue);
    If Depth = 0 Then
        return (0);
    Else
        BestScore := -infinity;
        Generate the move list for Player as per current board setup;
        For each legal move in the move list do
            Make the current move and get MoveScore;
            SubTreeCutoffValue := MoveScore - BestScore;
            Score := MoveScore - Search (enemy of Player, Depth - 1,
                                         SubTreeCutoffValue);
            UnMake the current move;
            If Score > BestScore then BestScore := Score;
            If BestScore >= CutoffValue then exit the For loop;
        End For;
        Return (BestScore);
    End If;
End CutoffSearch;
```

If any of the enemy's countermove scores are greater than the number of
points for the current player taking the enemy's piece minus the best score
found so far, then the net Score for the current player will be less than
the BestScore found so far, and BestScore will stay as it was.  The current
move can also be discarded for the 'equal to' case of above.  If the current
value of BestScore is negative infinity (which will be the case for the
first move in the movelist), then the cutoff value passed on will be
positive infinity, and no cutoffs will be possible in the subtree.  See the
Appendix for an example of a search with cutoffs.

There are a couple of cases in which the search tree will be pruned
extensively.  If all of the moves in the tree result in the same score, then
the 'equal to' case above will cause the search tree to be completely
minimized.  This case will evidence itself in the very beginning of the
game, when almost all of the moves have a zero score, since the pieces are
too far away for any takes to occur.  See the Appendix for an example of a
minimal cutoff search.  Also, as soon as the highest scoring move in a tree
node is found, all of the other moves will be cut off as soon as possible.
This gain can be especially significant if the highest scoring move is found
early in the root node.

With these changes implemented into the thinking routine of the program,
it ran much faster.  The three-ply Model 70 search time moved down to seven
seconds, and the four ply search time (which was now tolerable) was 45
seconds.

### 4.6. FINISHING TOUCHES

A few finishing touches were added to the thinking routine to make it
faster and smarter.  Since, as mentioned above, more cutoffs are possible
when the highest scoring move is encountered early in the search, and thus
less time is required for the search, a pre-scanning is called before the
main searching begins.  This pre-scanning routine, given the list of all
possible first moves, will perform a search of a smaller depth than the main
search and will sort the given moves in ascending order of the move's
pre-scan score.  This increases the chance of finding the best move earlier
in the main search.  An obviously very good move (like the second half of a
trade-off) will always be placed at the beginning of the main scan list.
This addition to the program has significantly reduced the scanning time for
the deeper searches.

Also, as mentioned above, in the case that there are a number of moves
that tie in score for best move, the move made was chosen randomly from the
tying moves.  A one-ply positional evaluation was added to break the tie in
most cases.  The move chosen will be the highest scoring one which results
in moving into the "best" position.  The position value is determined by
examining the move and the board.  For each enemy and friendly piece, the
total number of attacks and protects is tallied and a value is computed, and
this value is combined with the number of moves that the friendly side can
now possibly make.  Some other rules involving checking, castling, and
making developmental moves (pushing a pawn or taking a piece) are also
thrown in to make the player try to castle and to keep endless loops of
moves from occuring.  The sole intention of this routine was to break ties,
and the rules are completely ad hoc and were assigned values by a chess
non-expert.  A much better (and much more complex) set of rules and
assignment of values could be made by someone better qualified, and this
routine could be combined with the take-score of a move to make the position
evaluation n-ply along with the takes to make the play better.

Finally, the search routine was modified such that it would go up to two
plies deeper in scoring a line of play if all of the moves in that line were
takes.  This should make the computer react better to an 'arms race'
somewhere on the board.  Before it might stop searching too soon and not see
who comes out of the exchange better off.  There are probably many tricks
like the three mentioned here to make the computer think better, but we feel
that quite enough work was put into this area, given that the thinking was
not the focus of the project although it was certainly important.

## 5. PROGRAM FEATURES

The program that we have created not only plays the game of chess, it
incorporates many helpful and convient features.  Features like TakeBack and
UnTakeBack are helpful on those occasions when, even the best of us, make a
completely foolish move.  With the SetUp feature, a person is able to create
any board configuration he desires and play the game from there.  The File
System is especially useful for those long games when a break is needed.
These features and others make the program very easy to use and powerful at
the same time.

### 5.1. SETUP BOARD

The first of the program features that we will discuss is the SetUp. It
should be mentioned that the SetUp is completely intergrated with the other
features.  It is permitted for the user to temporarily suspend the playing
of the game and enter the SetUp.  He then can make any changes to the board
that he wants.  The consequence of this is that the moves prior to changing
the board are no longer accessible by Goto Move or TakeBack.

By selecting SetUp, the user is given a cursor which he can position
anywhere on the board he wishes.  To remove a specific piece from the board
the player should position the cursor on the piece and press the space bar.
To place a piece the user enters a key corresponding to that piece.  The
letter designations for all piece are as follows: (K = King, Q = Queen, B =
Bishop, R = Rook, N = Knight, and P = Pawn).  Once the type of piece is
determined, the color must be entered, either 'W' for white or 'B' for
black.  Placing a piece has a few restrictions.  For example a pawn may not
be placed on either the first or last row and each side must have exactly
one king.

In addition to asking the type and color of the piece, some pieces
require another bit of information.  If the king is placed on his starting
square then it must be known if the king has already been moved or not for
purposes of allowing/disallowing castling.  The same applies to a rook
placed in its original square.  For convenience pressing 'C' will remove all
pieces from the board and pressing 'D' will set up the default board. If the
user decides that he does not want to change the configuration, then by
pressing ESC the set up is abandoned and the old board is kept.

When the user has finished manipulating the board, by pressing RETURN the
changes are entered to memory.  The user is then asked for the move number
that the board set up corresponds.  Then the user is prompted to answer
whose turn comes next in the game.  Defaults are provided for both the move
number and the color of the next player to move.  The set up is now complete
and the game may continue.

### 5.2. TAKEBACK AND UNTAKEBACK

The TakeBack feature is really a bi-product of the computer move
algorithm, which makes all the possible moves and then takes them back in
order to find the best move.  It was decided that this would also be a
useful option for players.  It is not uncommon for some players to make a
move impulsively without looking at the consequences.  It is for these
players that the TakeBack was made accessible to the user.  Its operation is
very simple, the user presses the 'T' for TakeBack and the play of the game
is backed up one move.  This can be used to take back as many moves as
desired.  If it is decided by the user that the move that was made was
indeed the proper move, he can press the 'U' key and the move will be made
again and the board set up advanced to prior to the TakeBack.

The operation for the user seems very simple, but in fact there is a good
deal of work being done by the program.  The check status must be updated at
each move as well as the other status values in the display area.  If a
king, rook or pawn moves back to its original position then the flag for
determining if the piece has moved yet must be reset to again allow either
castling or the pawn to advance two squares.  The crucial aspect of the
TakeBack and UnTakeBack is that the environment must be identical to that
previous board set up and the environment must stay within the boundries of
the first move and the last move.

### 5.3. PLAYER SETTINGS

The user has control of several values that define a player.  When the
'P' is pressed, the user is prompted for these attributes of both the white
and black player.  First the user is asked to enter the name of the player
to be used on the display screen followed by the value of the elapsed time.
The elasped time works just like the standard chess clocks: your clock only
runs during your turn.  The format of the elapsed time is HH:MM:SS, which
represents the hours, minutes and seconds respectively. The next attribute
is the type of player, either Human or Computer.  The Look Ahead gives the
computer a basis of how well to play.  The range is from zero to nine which
represents how many moves into the future the computer will look before
deciding on a move.  Obviously, the greater values produce better moves but
also take much longer.  The final player attribute is the Positional
evaluation.  If this is turned on then a computer player will also count the
positional points in determining the best move.  All of these prompts have
there current values displayed and will be taken as the default if they are
not changed.

### 5.4. OPTIONS

There are also options that can be set that to not apply to players
rather the game as a whole.  These are accessible by pressing 'O' from the
main menu.  The first value to be set is the flash count.  This is the
number of times that the piece flashes before and after it moves.  The user
can also turn the sound on or off from this screen by indicating which at
the 'Sound' prompt.  Another aspect of the game that we thought the players
may want to have control over is en passent.  While this is entirely legal
move, more inexperienced players would probably prefer to play without en
passent.  The user also has control of the length of time that is waited
between moves while in the watch mode.  This can be adjusted to give as must
time or as little time between moves as desired. The final option is the
time limit.  By setting this value, the user can set the upper limit on the
clocks of the players.  When one of the clocks reach the time limit, the
player that has run out of time is declared the loser.

### 5.5. WATCH GAME

A feature put in merely for the fun of it is the Watch mode.  At any time
from the main menu the 'W' can be pressed and the moves that follow the
current move will be displayed until the end of the game.  Note that usually
there will be no moves following the current move.  In these cases the
entire game is displayed one move at a time starting at the first move.  To
recreate the game is really very straight forward.  All the moves must be
taken back until the beginning is reached.  From the beginning the moves are
made and displayed again by refering to the moves that were stored in the
game structure.

### 5.6. GO TO MOVE NUMBER

Another feature that may prove useful to the user is the Goto Move
function.  When the 'G' is pressed in the main menu, the user is asked for
the move number that corresponds to the board configuartion that the user
wishes to move to.  The color of the turn must also be entered so the exact
board configuation can be reached.  This is because that each move number
has both a white turn and a black turn.  For example, the first couple of
moves are as follows: move number one for white, move number one for black,
move number two for white, and so on.  The Goto feature was rather simple to
implement as once the move number is known, it is an easy task to either
TakeBack or UnTakeBack moves, without displaying the moves, until the
desired place is reached.

### 5.7. FILE SYSTEM

One of the most useful features is the File System.  With it games,
complete or incomplete, can be saved, loaded or printed.  Loading and saving
games is quite simple.  Since we were quite careful to keep all information
about the game in the GameType structure, it can merely be written or read
from the disk.  The user need only enter a new filename or use the default
that is provided and the game is loaded or saved.  A directory command is
also provided to allow the user to see a list of all games that have been
saved to disk.  Since a computer and disk drive is not always handy we have
given an option to print the game to either a file or to the printer.  When
the user selects the print option, he is asked to enter the filename to
contain the output.  If the user does wish to have a hard copy he should
enter PRN for the filename.

### 5.8. HINT

Another feature added to aid the inexperienced player is the Hint option.
When the player needs some help in deciding the best move, he can press 'H'
and have the computer suggest a move.  The computer will use, as a basis for
how far to look, the look ahead value stored in the PlayerType structure.
The result, after doing the search for the best move, will be printed to the
display area.  The player must still make either the suggested move or one
that he feels is better.

### 5.9. ON-LINE HELP

The entire system is very easy to use and displays instructions almost
everywhere although beginners may some problems.  For this reason an on-line
help feature has been incorporated.  This help does not explain the rules of
chess as that was not the purpose of this project.  It does, however,
explain how to use the program and should answer any problems that a user
would have.  The help is made up of several pages of instructions.  You
should use the Up and Down page keys to turn the pages and press ESC when
you want to leave the help facility and resume the program.

### 5.10. HIGH RESOLUTION GRAPHICS

The high resolution graphics add a whole dimension of realism to a chess
game over a character based display with K for king, etc.  Also, since Turbo
Pascal provides a complete graphics unit, our program implements a high
resolution display of the main game screen.  Since the VGA graphics card is
the standard for the new and popular PS/2 line of computers, the graphics
were designed for this type of display.  The VGA display has a one to one
pixel aspect ratio.

The first step was to divide the screen up into its major sections and
then see how many pixels the piece images would be, which turned out to be
52 pixels squared.  Then, a text file containing all of the images
represented by astrisks and spaces was prepared, where these images were
borrowed from the Sargon II cartridge for the VIC-20 computer and enlarged
and sand-blasted to fit the VGA piece size.  A small program was written to
read the text file, place the images on the screen over the three colors of
background (dark, light, and cursor), and then these images were lifted from
the screen, put into a structure, and written to a disk file.

The main program reads this binary pixel image file into an array when it
starts up.  Then, the routine DisplaySquare is called with the parameters
Row, Col, and Cursor (yes or no).  Based on these parameters, it figures out
whether the background of the square should be dark, light, or the cursor
color.  It then looks at the internal Board matrix to see which piece image
and piece color to display, and these three values are used as subscripts
into the pixel image array and puts the image onto the screen.  The
upper-left pixel location of where to put the object is obtained by
multiplying the row and column by 52 and adding the offset of the board from
the screen edges.

The procedure to fill a rectangular area on the screen with a given
color, and all of the text sizes and fonts shown on the screen are provided
by Turbo Pascal.  The only difficulty was in figuring out what to say and
exactly where to put it.  A number of pixel location constants are declared
at the beginning of the program.  The CenterText procedure does most of the
work of putting text in the conversation area, and the DisplayInstructions
procedure displays all of the instructions at the bottom of the screen.

## 6. PROGRAM LIMITATIONS

The greatest limitation of the program is the extremely simple evaluation
of a given move.  The score is determined exclusively by the number of value
points assigned to the piece that is taken.  If no piece is taken then no
points are awarded.  As implemented, the positional evaluation plays only a
small roll in selecting a move.  A better positional evaluator, which takes
into account such things as attack strategy and special situations requiring
special actions, etc. should be combined with a better capture value to
calculate the value at all nodes in the search tree.  A better capture value
would take into account the fact that the material value of the piece varies
with the stage and balance of the game.  The combination of material and
position values should be consistent with the strategy and circumstances of
the game.  Of course this was beyond the scope of this project (not to
mention our own skill and knowledge of chess).  This could be a future
enhancement.

Another deficiency is with the search tree algorithm itself.  Since it
only searches to a certain depth, it may attempt to push an inevitable loss
out of its sight by sacrificing a less valuable piece to 'save' the greater
valued one.  This is known as the Horizon Effect and is an inevitable
consequence of the limited search.  The only solution is to increase the
search depth.  This often would only serve to move the horizon a little
father down the road.  The solution to this problem is also beyond the scope
of this project.

## 7. CONCLUSION

We have succeeded in designing and implementing a working chess program.
The program allows both human and computer players and also offers a number
of powerful features to manipulate the game.  The computer movement portion
presented the greatest challenge and yet proved to be by far the most
interesting.  The only deficiencies of the program lie in the strategy of
the computers movement.  The rest of the program was very straight forward
to implement, with only a few temporary setbacks.
