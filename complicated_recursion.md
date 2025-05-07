## A More Complicated Example of Recursion
Anyone really keen to learn about recursion (I suspect very few - if any) might wish to take a look at this code.
It was written quite quickly, so may not be as elegant as it could be, but it does seem to work.

Copy/paste the following into a file called ``noughtcross.md``, and type ``noughtcross``to run it:

```matlab
function noughtcross
% Noughts and crosses
% There are no input or output arguments

    % Set up an empty board
    X = char(zeros(3) + ' ');

    % Lets get going...
    while(any(any(X==' ')))

        % User's go
        X = get_next_x(X);

        % Show the board and finish if the game is done
        if show_board(X)~=0 || ~any(X(:)==' ')
            break;
        end

        % Computer's go
        X = get_next_o(X);

        % Show the board and finish if the game is done
        if show_board(X)~=0 || ~any(X(:)==' ')
            break;
        end
    end
end


function t = show_board(X)

    % Display the board
    s = ('   ')';
    v = ('|||')';
    disp([('  123 ')' ['  1 2 3  '; '/-------\'; v s X(:,1) s X(:,2) s X(:,3) s v; '\-------/']])

    % Get the score
    f = score(X);

    if f<0
        disp('You (X) win.')
        t = true;
    elseif f>0
        disp('Computer (O) wins.')
        t = true;
    else
        % Check for a draw
        if all(sum(X=='o',1)>0 & sum(X=='x',1)>0) && ...
           all(sum(X=='o',2)>0 & sum(X=='x',2)>0) && ...
           sum(diag(X)=='x')>0 && sum(diag(X)=='o')>0 && ...
           sum(diag(flipud(X))=='x')>0 && sum(diag(flipud(X))=='o')>0
            disp('Draw.')
            t = true;
        else
            % Keep on playing
            t = false;
        end
    end
end


function X = get_next_x(X)
    % User enters their move and the board is updated

    i = input('Row: ');
    while length(i)~=1 || rem(i,1)~=0 || i<1 || i>3 || sum(X(i,:)==' ')==0
        i = input('Row: ');
    end

    j = input('Col: ');
    while length(j)~=1 || rem(j,1)~=0 || j<1 || j>3 || X(i,j)~=' '
        j = input('Col: ');
    end
    X(i,j) = 'x';
end


function f = score(X)
    % Return a score of either:
    % -1 - User (X) wins
    % +1 - Computer (O) wins
    % 0  - Nobody has won yet
    x_win = any(sum(X=='x',1)==3) || any(sum(X=='x',2)==3) || sum(diag(X)=='x')==3 || sum(diag(flipud(X))=='x')==3;
    o_win = any(sum(X=='o',1)==3) || any(sum(X=='o',2)==3) || sum(diag(X)=='o')==3 || sum(diag(flipud(X))=='o')==3;
    f     = o_win - x_win;
end


function X = get_next_o(X)
% Update the board with the best move for the computer

    f_best = -Inf;
    i_best = 1;
    j_best = 1;

    % Scan through possible positions to find the best
    for i=1:3
        for j=1:3
            if X(i,j)==' ' % Available place to put an "o"

                % Board configuration to try
                Xt      = X;
                Xt(i,j) = 'o';

                % Compute the utility for this position
                f       = search(Xt,'x');

                if f > f_best % Best utility function so far
                    i_best = i;
                    j_best = j;
                    f_best = f;
                end
            end
        end
    end

    % Make the move with the most likely win
    X(i_best,j_best) = 'o';
end


function f = search(X, piece)
    % Return the score for this move
    f = score(X);
    if f ~= 0
        % If a winning/losing position then done
        return % This means "leave the function early"
    end

    % Work out the piece for the next player
    if piece=='x'
        next_piece = 'o';
    else
        next_piece = 'x';
    end

    % Scan through available positions and tot up the probable score
    ind = find(X==' ');
    for i = ind'
        Xt    = X;
        Xt(i) = piece;
        % Note the recursion in the next line
        f     = f + search(Xt, next_piece)/length(ind);
    end
end
```
