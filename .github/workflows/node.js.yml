import React, { useState, useEffect, useRef } from 'react';

function MazeGame() {
  // Define the difficulty settings including a new "extreme" level.
  const difficulties = {
    easy: { rows: 11, cols: 11, coinRate: 0.2, cellClass: "w-8 h-8", npcCount: 0 },
    medium: { rows: 15, cols: 15, coinRate: 0.15, cellClass: "w-6 h-6", npcCount: 1 },
    hard: { rows: 21, cols: 21, coinRate: 0.1, cellClass: "w-4 h-4", npcCount: 2 },
    extreme: { rows: 31, cols: 31, coinRate: 0.07, cellClass: "w-3 h-3", npcCount: 3 }
  };

  // Game configuration toggles from menu.
  const [difficulty, setDifficulty] = useState('easy');
  const [trapsEnabled, setTrapsEnabled] = useState(false);
  const [npcsEnabled, setNpcsEnabled] = useState(false);

  // Derived settings based on selected difficulty.
  const { rows, cols, coinRate, cellClass, npcCount } = difficulties[difficulty];

  // Positions
  const startPos = { row: 1, col: 1 };

  // Game states.
  const [mazeData, setMazeData] = useState([]);
  const [finishPos, setFinishPos] = useState({ row: rows - 2, col: cols - 2 });
  const [playerPos, setPlayerPos] = useState(startPos);
  const [coinPositions, setCoinPositions] = useState([]); // coins array [{row, col},...]
  const [trapPositions, setTrapPositions] = useState([]); // traps array [{row, col},...]
  const [npcPositions, setNpcPositions] = useState([]); // NPCs array [{row, col},...]
  const [coinCount, setCoinCount] = useState(0);
  const [gameWon, setGameWon] = useState(false);
  const [gameLost, setGameLost] = useState(false);

  // Menu state.
  const [showMenu, setShowMenu] = useState(true);

  // Helper function: Shuffle array (Fisher-Yates).
  const shuffleArray = (array) => {
    const arr = array.slice();
    for (let i = arr.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [arr[i], arr[j]] = [arr[j], arr[i]];
    }
    return arr;
  };

  // Generate a maze using recursive backtracking algorithm.
  // Maze: 1 = wall, 0 = path.
  const generateMaze = (mazeRows, mazeCols) => {
    const maze = Array.from({ length: mazeRows }, () =>
      Array.from({ length: mazeCols }, () => 1)
    );
    const stack = [];
    maze[startPos.row][startPos.col] = 0;
    stack.push({ row: startPos.row, col: startPos.col });

    while (stack.length > 0) {
      const current = stack[stack.length - 1];
      const { row, col } = current;
      const directions = shuffleArray([
        { dRow: -2, dCol: 0 },
        { dRow: 0, dCol: 2 },
        { dRow: 2, dCol: 0 },
        { dRow: 0, dCol: -2 }
      ]);
      let moved = false;
      for (let direction of directions) {
        const newRow = row + direction.dRow;
        const newCol = col + direction.dCol;
        if (
          newRow > 0 && newRow < mazeRows - 1 &&
          newCol > 0 && newCol < mazeCols - 1 &&
          maze[newRow][newCol] === 1
        ) {
          // Remove wall between cells.
          maze[row + Math.floor(direction.dRow / 2)][col + Math.floor(direction.dCol / 2)] = 0;
          maze[newRow][newCol] = 0;
          stack.push({ row: newRow, col: newCol });
          moved = true;
          break;
        }
      }
      if (!moved) {
        stack.pop();
      }
    }
    return maze;
  };

  // Use BFS to find the farthest reachable cell from start.
  const findFarthestCell = (maze, start) => {
    const mazeRows = maze.length;
    const mazeCols = maze[0].length;
    const visited = Array.from({ length: mazeRows }, () =>
      Array.from({ length: mazeCols }, () => false)
    );
    const queue = [];
    queue.push({ ...start, dist: 0 });
    visited[start.row][start.col] = true;
    let farthest = { row: start.row, col: start.col, dist: 0 };

    while (queue.length > 0) {
      const current = queue.shift();
      const { row, col, dist } = current;
      if (dist > farthest.dist) {
        farthest = { row, col, dist };
      }
      const directions = [
        { dRow: -1, dCol: 0 },
        { dRow: 1, dCol: 0 },
        { dRow: 0, dCol: -1 },
        { dRow: 0, dCol: 1 }
      ];
      for (let { dRow, dCol } of directions) {
        const newRow = row + dRow;
        const newCol = col + dCol;
        if (
          newRow >= 0 && newRow < mazeRows &&
          newCol >= 0 && newCol < mazeCols &&
          !visited[newRow][newCol] &&
          maze[newRow][newCol] === 0
        ) {
          visited[newRow][newCol] = true;
          queue.push({ row: newRow, col: newCol, dist: dist + 1 });
        }
      }
    }
    return { row: farthest.row, col: farthest.col };
  };

  // Generate coins for the maze. Excludes start and finish positions.
  const generateCoins = (maze, rate, start, finish) => {
    const coins = [];
    for (let r = 0; r < maze.length; r++) {
      for (let c = 0; c < maze[0].length; c++) {
        if (maze[r][c] === 0) {
          if ((r === start.row && c === start.col) || (r === finish.row && c === finish.col)) {
            continue;
          }
          if (Math.random() < rate) {
            coins.push({ row: r, col: c });
          }
        }
      }
    }
    return coins;
  };

  // Generate traps for the maze. Excludes start, finish and coin positions.
  const generateTraps = (maze, trapRate, start, finish, coins) => {
    const traps = [];
    for (let r = 0; r < maze.length; r++) {
      for (let c = 0; c < maze[0].length; c++) {
        if (maze[r][c] === 0) {
          if ((r === start.row && c === start.col) ||
              (r === finish.row && c === finish.col) ||
              coins.some((coin) => coin.row === r && coin.col === c)) {
            continue;
          }
          if (Math.random() < trapRate) {
            traps.push({ row: r, col: c });
          }
        }
      }
    }
    return traps;
  };

  // Generate NPCs: Place them randomly on open path cells (excluding start and finish).
  const generateNpcs = (maze, count, start, finish) => {
    const npcs = [];
    const candidates = [];
    for (let r = 0; r < maze.length; r++) {
      for (let c = 0; c < maze[0].length; c++) {
        if (
          maze[r][c] === 0 &&
          !(r === start.row && c === start.col) &&
          !(r === finish.row && c === finish.col)
        ) {
          candidates.push({ row: r, col: c });
        }
      }
    }
    const shuffled = shuffleArray(candidates);
    for (let i = 0; i < count && i < shuffled.length; i++) {
      npcs.push(shuffled[i]);
    }
    return npcs;
  };

  // Regenerate the game: creates maze, coins, traps, and NPCs (if enabled).
  const regenerateGame = () => {
    const newMaze = generateMaze(rows, cols);
    const newFinish = findFarthestCell(newMaze, startPos);
    newMaze[newFinish.row][newFinish.col] = 0; // ensure finish is open
    const coins = generateCoins(newMaze, coinRate, startPos, newFinish);
    const traps = trapsEnabled ? generateTraps(newMaze, 0.05, startPos, newFinish, coins) : [];
    const npcs = npcsEnabled ? generateNpcs(newMaze, npcCount, startPos, newFinish) : [];
    setMazeData(newMaze);
    setFinishPos(newFinish);
    setPlayerPos(startPos);
    setCoinCount(0);
    setGameWon(false);
    setGameLost(false);
    setCoinPositions(coins);
    setTrapPositions(traps);
    setNpcPositions(npcs);
  };

  // Initialize/reinitialize game when difficulty or menu state changes.
  useEffect(() => {
    if (!showMenu) {
      regenerateGame();
    }
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, [difficulty, showMenu]);

  // Check if move is valid: within bounds and not a wall.
  const canMoveTo = (row, col) => {
    return (
      row >= 0 &&
      row < rows &&
      col >= 0 &&
      col < cols &&
      mazeData[row][col] === 0
    );
  };

  // Check for collisions with traps or NPCs.
  const checkForHazards = (pos) => {
    // Check traps.
    if (trapPositions.some((trap) => trap.row === pos.row && trap.col === pos.col)) {
      return "trap";
    }
    // Check NPC collision.
    if (npcPositions.some((npc) => npc.row === pos.row && npc.col === pos.col)) {
      return "npc";
    }
    return null;
  };

  // Handle player movement.
  const movePlayer = (direction) => {
    if (gameWon || gameLost) return;
    let newRow = playerPos.row;
    let newCol = playerPos.col;
    if (direction === 'up') newRow -= 1;
    else if (direction === 'down') newRow += 1;
    else if (direction === 'left') newCol -= 1;
    else if (direction === 'right') newCol += 1;

    if (canMoveTo(newRow, newCol)) {
      const newPos = { row: newRow, col: newCol };
      setPlayerPos(newPos);

      // If player collects coin.
      const coinIndex = coinPositions.findIndex(
        (coin) => coin.row === newRow && coin.col === newCol
      );
      if (coinIndex !== -1) {
        setCoinCount(coinCount + 1);
        setCoinPositions(coinPositions.filter((_, idx) => idx !== coinIndex));
      }

      // Check if player stepped on a hazard.
      const hazard = checkForHazards(newPos);
      if (hazard) {
        setGameLost(true);
      }
      // Check if reached finish (ensure finish cell is hazard-free).
      if (newRow === finishPos.row && newCol === finishPos.col) {
        setGameWon(true);
      }
    }
  };

  // NPC random movement: every 1 second, move each NPC randomly.
  const npcMoveInterval = useRef(null);
  useEffect(() => {
    if (npcsEnabled && !gameWon && !gameLost) {
      npcMoveInterval.current = setInterval(() => {
        setNpcPositions((prevNpcs) => {
          return prevNpcs.map((npc) => {
            const directions = [
              { dRow: -1, dCol: 0 },
              { dRow: 1, dCol: 0 },
              { dRow: 0, dCol: -1 },
              { dRow: 0, dCol: 1 }
            ];
            const shuffled = shuffleArray(directions);
            for (let { dRow, dCol } of shuffled) {
              const newRow = npc.row + dRow;
              const newCol = npc.col + dCol;
              if (canMoveTo(newRow, newCol)) {
                return { row: newRow, col: newCol };
              }
            }
            return npc;
          });
        });
      }, 1000);
      return () => clearInterval(npcMoveInterval.current);
    }
  }, [npcsEnabled, gameWon, gameLost, mazeData]);

  // Check collision between player and NPC after every npcPositions update.
  useEffect(() => {
    if (npcPositions.some((npc) => npc.row === playerPos.row && npc.col === playerPos.col)) {
      setGameLost(true);
    }
  }, [npcPositions, playerPos]);

  // Keyboard controls.
  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.key === 'ArrowUp') {
        e.preventDefault();
        movePlayer('up');
      } else if (e.key === 'ArrowDown') {
        e.preventDefault();
        movePlayer('down');
      } else if (e.key === 'ArrowLeft') {
        e.preventDefault();
        movePlayer('left');
      } else if (e.key === 'ArrowRight') {
        e.preventDefault();
        movePlayer('right');
      }
    };
    window.addEventListener('keydown', handleKeyDown);
    return () => window.removeEventListener('keydown', handleKeyDown);
  }, [playerPos, gameWon, gameLost, mazeData, coinPositions, trapPositions, npcPositions]);

  // Render a single cell.
  const renderCell = (rowIndex, colIndex) => {
    let cellType = "";
    if (mazeData[rowIndex][colIndex] === 1) {
      cellType = "wall";
    } else {
      cellType = "path";
    }
    if (rowIndex === playerPos.row && colIndex === playerPos.col) {
      cellType = "player";
    } else if (rowIndex === finishPos.row && colIndex === finishPos.col) {
      cellType = "finish";
    }

    const baseClasses =
      "border border-gray-300 flex items-center justify-center";
    let cellColorClasses = "";
    if (cellType === "wall") cellColorClasses = "bg-gray-700";
    else if (cellType === "player") cellColorClasses = "bg-green-500";
    else if (cellType === "finish") cellColorClasses = "bg-yellow-400";
    else cellColorClasses = "bg-white";

    // Render coin if available on a plain path cell.
    const hasCoin =
      coinPositions.some(
        (coin) => coin.row === rowIndex && coin.col === colIndex
      ) && cellType === "path";
    // Render trap icon if traps enabled.
    const hasTrap =
      trapsEnabled &&
      trapPositions.some(
        (trap) => trap.row === rowIndex && trap.col === colIndex
      ) &&
      cellType === "path";
    // Render NPC if present.
    const hasNPC =
      npcsEnabled &&
      npcPositions.some(
        (npc) => npc.row === rowIndex && npc.col === colIndex
      ) &&
      cellType === "path";

    return (
      <div
        key={`${rowIndex}-${colIndex}`}
        className={`${baseClasses} ${cellClass} ${cellColorClasses}`}
      >
        {hasCoin && <span role="img" aria-label="coin" className="text-xs">💰</span>}
        {hasTrap && <span role="img" aria-label="trap" className="text-xs">⚠️</span>}
        {hasNPC && <span role="img" aria-label="npc" className="text-xs">👾</span>}
      </div>
    );
  };

  // Menu and controls interface.
  if (showMenu) {
    return (
      <div className="min-h-screen bg-gray-100 flex flex-col items-center justify-center p-4">
        <h1 className="text-2xl font-bold mb-4">Лабиринт с монетами и ловушками</h1>
        <div className="bg-white p-4 rounded shadow text-center mb-4">
          <p className="mb-2">
            Цель игры – собрать монеты, избегать ловушек и бродячих персонажей, а также дойти до жёлтого квадрата (конца лабиринта).
          </p>
          <p className="mb-2">
            Управление: используйте стрелки на клавиатуре или кнопки на экране.
          </p>
          <p className="mb-2">Выберите уровень сложности и настройте дополнительные опции.</p>
          <p className="mb-2 text-sm text-gray-600">
            На уровне "Extreme" лабиринт будет огромным, но масштабированным под размер экрана.
          </p>
        </div>
        <div className="flex gap-2 mb-4 flex-wrap justify-center">
          {["easy", "medium", "hard", "extreme"].map((level) => (
            <button
              key={level}
              onClick={() => setDifficulty(level)}
              className={`px-3 py-1 rounded ${difficulty === level ? 'bg-blue-600 text-white' : 'bg-gray-300 text-gray-800'} focus:outline-none`}
            >
              {level === "easy" ? "Лёгкий" : 
               level === "medium" ? "Средний" : 
               level === "hard" ? "Сложный" : "Extreme"}
            </button>
          ))}
        </div>
        <div className="flex gap-4 mb-4 flex-wrap justify-center">
          <label className="flex items-center gap-1">
            <input
              type="checkbox"
              checked={trapsEnabled}
              onChange={() => setTrapsEnabled(!trapsEnabled)}
              className="form-checkbox"
            />
            Ловушки
          </label>
          <label className="flex items-center gap-1">
            <input
              type="checkbox"
              checked={npcsEnabled}
              onChange={() => setNpcsEnabled(!npcsEnabled)}
              className="form-checkbox"
            />
            Блуждающие персонажи
          </label>
        </div>
        <button
          onClick={() => setShowMenu(false)}
          className="bg-purple-500 text-white px-4 py-2 rounded focus:outline-none"
        >
          Начать игру
        </button>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 flex flex-col items-center p-4">
      <header className="w-full flex flex-col items-center mb-4">
        <h1 className="text-2xl font-bold">Лабиринт с монетами и ловушками</h1>
        <div className="flex gap-2 mt-2 flex-wrap justify-center">
          {["easy", "medium", "hard", "extreme"].map((level) => (
            <button
              key={level}
              onClick={() => setDifficulty(level)}
              className={`px-3 py-1 rounded ${difficulty === level ? 'bg-blue-600 text-white' : 'bg-gray-300 text-gray-800'} focus:outline-none`}
            >
              {level === "easy" ? "Лёгкий" : 
               level === "medium" ? "Средний" : 
               level === "hard" ? "Сложный" : "Extreme"}
            </button>
          ))}
        </div>
        <p className="mt-2 font-semibold">Собрано монет: {coinCount}</p>
        {gameLost && (
          <p className="mt-2 text-red-600 font-bold">Игра окончена! Вы попали в ловушку или столкнулись с персонажем.</p>
        )}
        {gameWon && (
          <p className="mt-2 text-green-600 font-bold">Поздравляем! Вы дошли до конца лабиринта!</p>
        )}
      </header>

      <div className="overflow-auto flex justify-center max-w-full">
        {mazeData.length > 0 && (
          <div 
            className="grid" 
            style={{ gridTemplateColumns: `repeat(${cols}, 1fr)` }}
          >
            {mazeData.map((row, rowIndex) =>
              row.map((_, colIndex) => renderCell(rowIndex, colIndex))
            )}
          </div>
        )}
      </div>

      <div className="mt-6 flex flex-col items-center">
        <p className="mb-2 font-semibold">Управление</p>
        <div className="flex flex-col items-center">
          <button 
            onClick={() => movePlayer('up')} 
            className="bg-blue-500 text-white px-4 py-2 rounded mb-2 focus:outline-none"
          >
            ↑
          </button>
          <div className="flex">
            <button 
              onClick={() => movePlayer('left')} 
              className="bg-blue-500 text-white px-4 py-2 rounded mr-2 focus:outline-none"
            >
              ←
            </button>
            <button 
              onClick={() => movePlayer('down')} 
              className="bg-blue-500 text-white px-4 py-2 rounded mr-2 focus:outline-none"
            >
              ↓
            </button>
            <button 
              onClick={() => movePlayer('right')} 
              className="bg-blue-500 text-white px-4 py-2 rounded focus:outline-none"
            >
              →
            </button>
          </div>
          <p className="mt-2 text-sm text-gray-600">
            Используйте стрелки на клавиатуре или кнопки на экране
          </p>
        </div>
      </div>

      <div className="mt-6 flex gap-4 flex-wrap justify-center">
        <button 
          onClick={regenerateGame}
          className="bg-purple-500 text-white px-4 py-2 rounded focus:outline-none"
        >
          Новая игра
        </button>
        <button 
          onClick={() => setShowMenu(true)}
          className="bg-gray-500 text-white px-4 py-2 rounded focus:outline-none"
        >
          Меню
        </button>
      </div>
    </div>
  );
}

export default MazeGame;
