<script lang="ts">
    import { fade, scale } from 'svelte/transition';

    // Ocean Professional theme tokens (synced with app.css variables)
    const THEME = {
        primary: '#2563EB',
        secondary: '#F59E0B',
        success: '#F59E0B',
        error: '#EF4444',
        bg: 'var(--op-bg)',
        surface: 'var(--op-surface)',
        text: 'var(--op-text)',
        textMuted: 'var(--op-text-muted)',
        ring: 'var(--op-ring)',
        grid: 'var(--op-grid)',
    };

    type Cell = 'X' | 'O' | '';

    // PUBLIC_INTERFACE
    export function createEmptyBoard(): Cell[] {
        /** Create a new empty tic tac toe board with 9 cells. */
        return Array<Cell>(9).fill('');
    }

    let board: Cell[] = createEmptyBoard();
    let currentPlayer: 'X' | 'O' = 'X';
    let gameOver = false;
    let winner: 'X' | 'O' | 'Draw' | null = null;
    let lastWinLine: number[] | null = null;

    // PUBLIC_INTERFACE
    export function resetGame() {
        /** Reset the game to its initial state. */
        board = createEmptyBoard();
        currentPlayer = 'X';
        gameOver = false;
        winner = null;
        lastWinLine = null;
    }

    // PUBLIC_INTERFACE
    export function handleCellClick(index: number) {
        /** Handle a user clicking a cell, place a mark if valid and update state. */
        if (gameOver || board[index]) return;
        board = board.map((c, i) => (i === index ? currentPlayer : c));

        const result = checkWinner(board);
        if (result) {
            gameOver = true;
            if (result.type === 'win') {
                winner = result.player;
                lastWinLine = result.line;
            } else {
                winner = 'Draw';
            }
        } else {
            currentPlayer = currentPlayer === 'X' ? 'O' : 'X';
        }
    }

    // PUBLIC_INTERFACE
    export function checkWinner(b: Cell[]): { type: 'win'; player: 'X' | 'O'; line: number[] } | { type: 'draw' } | null {
        /**
         * Determine if the provided board has a winner or a draw.
         * Returns:
         * - { type: 'win', player, line } when a player wins
         * - { type: 'draw' } when all cells are filled with no winner
         * - null when the game should continue
         */
        const LINES = [
            [0, 1, 2],
            [3, 4, 5],
            [6, 7, 8],
            [0, 3, 6],
            [1, 4, 7],
            [2, 5, 8],
            [0, 4, 8],
            [2, 4, 6]
        ];

        for (const line of LINES) {
            const [a, b2, c] = line;
            if (b[a] && b[a] === b[b2] && b[a] === b[c]) {
                return { type: 'win', player: b[a] as 'X' | 'O', line };
            }
        }

        if (b.every((cell) => cell)) {
            return { type: 'draw' };
        }
        return null;
    }

    // Derived helpers
    $: statusText = (() => {
        if (winner === 'Draw') return 'It’s a draw. Try again!';
        if (winner === 'X' || winner === 'O') return `Player ${winner} wins!`;
        return `Player ${currentPlayer}'s turn`;
    })();

    $: statusAccent = (() => {
        if (winner === 'Draw') return THEME.secondary;
        if (winner === 'X' || winner === 'O') return THEME.success;
        return THEME.primary;
    })();

    function isWinningIndex(i: number) {
        return !!lastWinLine?.includes(i);
    }
</script>

<svelte:head>
    <title>Tic Tac Toe — Ocean Professional</title>
    <meta name="description" content="Play Tic Tac Toe with a modern, minimalist Ocean Professional theme." />
</svelte:head>

<div class="page" in:fade={{ duration: 250 }}>
    <div class="card" style="--status-accent: {statusAccent}">
        <header class="header" in:fade={{ duration: 250 }}>
            <div class="title-wrap">
                <h1>Tic Tac Toe</h1>
                <p>Modern • Ocean Professional</p>
            </div>

            <div class="controls">
                <button class="btn ghost" on:click={resetGame} aria-label="Reset game" title="Reset game">
                    <svg width="18" height="18" viewBox="0 0 24 24" fill="none" aria-hidden="true">
                        <path d="M12 5V1L7 6l5 5V7c3.31 0 6 2.69 6 6a6 6 0 0 1-6 6 6 6 0 0 1-5.65-4H4.26A8 8 0 0 0 12 23c4.42 0 8-3.58 8-8s-3.58-8-8-8Z" fill="currentColor" />
                    </svg>
                    Reset
                </button>
            </div>
        </header>

        <section class="status" in:fade={{ duration: 200 }}>
            <div class="chip" style="--chip-accent: {statusAccent}">
                {statusText}
            </div>
            {#if !gameOver}
                <div class="turn">
                    <span class="label">Current:</span>
                    <span class="badge {currentPlayer === 'X' ? 'x' : 'o'}" in:scale>{currentPlayer}</span>
                </div>
            {/if}
        </section>

        <div class="board" role="grid" aria-label="Tic Tac Toe board">
            {#each board as cell, i (i)}
                <button
                    class="cell {isWinningIndex(i) ? 'win' : ''}"
                    role="gridcell"
                    aria-label={"Cell " + (i + 1)}
                    aria-disabled={!!cell || gameOver}
                    disabled={!!cell || gameOver}
                    on:click={() => handleCellClick(i)}
                    in:scale={{ duration: 100 }}
                >
                    {#if cell}
                        <span class="mark {cell}" in:scale={{ duration: 120 }}>{cell}</span>
                    {/if}
                </button>
            {/each}
        </div>

        <footer class="footer" in:fade={{ duration: 200 }}>
            <button class="btn primary" on:click={resetGame} aria-label="Play again">
                {gameOver ? 'Play again' : 'Reset'}
            </button>
        </footer>
    </div>
</div>

<style>
    /* Container layout */
    .page {
        width: 100%;
        max-width: 560px;
        padding: 1rem;
        display: flex;
        align-items: center;
        justify-content: center;
    }

    .card {
        width: 100%;
        background: var(--op-surface);
        border-radius: 16px;
        box-shadow:
            0 10px 15px -3px rgba(17, 24, 39, 0.08),
            0 4px 6px -4px rgba(17, 24, 39, 0.06);
        overflow: hidden;
        border: 1px solid var(--op-surface-border);
        transition: transform 180ms ease, box-shadow 180ms ease;
        backdrop-filter: saturate(110%) blur(2px);
    }

    .card:hover {
        transform: translateY(-1px);
        box-shadow:
            0 20px 25px -5px rgba(17, 24, 39, 0.10),
            0 8px 10px -6px rgba(17, 24, 39, 0.08);
    }

    /* Header */
    .header {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 1rem 1rem 0.25rem 1rem;
    }

    .title-wrap h1 {
        font-size: 1.25rem;
        line-height: 1.2;
        margin: 0;
        color: var(--op-text);
        letter-spacing: 0.2px;
    }
    .title-wrap p {
        margin: 0.25rem 0 0 0;
        color: var(--op-text-muted);
        font-size: 0.9rem;
    }

    .controls {
        display: flex;
        gap: 0.5rem;
    }

    /* Status section */
    .status {
        padding: 0.75rem 1rem 0.5rem 1rem;
        display: flex;
        align-items: center;
        justify-content: space-between;
        gap: 0.75rem;
    }

    .chip {
        background: linear-gradient(135deg, color-mix(in oklab, var(--status-accent) 16%, transparent), transparent);
        color: var(--op-text);
        border: 1px solid color-mix(in oklab, var(--status-accent) 35%, transparent);
        padding: 0.55rem 0.75rem;
        border-radius: 10px;
        font-size: 0.95rem;
        white-space: nowrap;
    }

    .turn {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        color: var(--op-text-muted);
    }

    .badge {
        display: inline-flex;
        align-items: center;
        justify-content: center;
        width: 28px;
        height: 28px;
        border-radius: 8px;
        font-weight: 700;
        color: white;
        letter-spacing: 0.5px;
        text-shadow: 0 1px 0 rgba(0,0,0,0.08);
    }

    .badge.x {
        background: var(--op-primary);
        box-shadow: 0 6px 12px -4px color-mix(in oklab, var(--op-primary) 40%, transparent);
    }

    .badge.o {
        background: var(--op-secondary);
        box-shadow: 0 6px 12px -4px color-mix(in oklab, var(--op-secondary) 40%, transparent);
    }

    /* Board */
    .board {
        display: grid;
        grid-template-columns: repeat(3, 1fr);
        gap: 10px;
        padding: 1rem;
        aspect-ratio: 1/1;
    }

    .cell {
        background:
            linear-gradient(180deg, rgba(37, 99, 235, 0.04), transparent 45%),
            var(--op-surface);
        border: 1px solid var(--op-grid);
        border-radius: 14px;
        position: relative;
        box-shadow: inset 0 1px 0 rgba(255,255,255,0.4);
        display: flex;
        align-items: center;
        justify-content: center;
        transition: transform 120ms ease, box-shadow 120ms ease, border-color 120ms ease, background 180ms ease, opacity 120ms ease;
        cursor: pointer;
        min-height: 84px;
        user-select: none;
        -webkit-tap-highlight-color: transparent;
        touch-action: manipulation;
    }

    .cell:hover:not(:disabled) {
        transform: translateY(-1px);
        box-shadow:
            inset 0 1px 0 rgba(255,255,255,0.5),
            0 10px 18px -12px rgba(37, 99, 235, 0.35);
        border-color: color-mix(in oklab, var(--op-primary) 25%, var(--op-grid));
    }

    .cell:active:not(:disabled) {
        transform: translateY(0);
        box-shadow:
            inset 0 1px 0 rgba(255,255,255,0.5),
            0 8px 14px -12px rgba(37, 99, 235, 0.35);
    }

    .cell:disabled {
        cursor: default;
        opacity: 0.98;
    }

    .cell.win {
        border-color: color-mix(in oklab, var(--op-secondary) 50%, var(--op-grid));
        box-shadow: 0 14px 20px -12px rgba(245, 158, 11, 0.45);
        background:
            linear-gradient(145deg, rgba(245, 158, 11, 0.10), transparent 55%),
            var(--op-surface);
    }

    .mark {
        font-size: clamp(2.5rem, 7.8vw, 4rem);
        font-weight: 800;
        line-height: 1;
        letter-spacing: 1px;
        text-shadow: 0 1px 0 rgba(255,255,255,0.35);
    }

    .mark.X {
        color: var(--op-primary);
        filter: drop-shadow(0 6px 10px rgba(37,99,235,0.35));
    }

    .mark.O {
        color: var(--op-secondary);
        filter: drop-shadow(0 6px 10px rgba(245,158,11,0.35));
    }

    /* Footer */
    .footer {
        padding: 0.5rem 1rem 1rem 1rem;
        display: flex;
        justify-content: center;
    }

    /* Buttons */
    .btn {
        border: 1px solid var(--op-button-border);
        background: linear-gradient(180deg, rgba(255,255,255,0.6), rgba(255,255,255,0.3));
        color: var(--op-text);
        padding: 0.6rem 0.9rem;
        border-radius: 12px;
        font-weight: 600;
        letter-spacing: 0.25px;
        cursor: pointer;
        transition: transform 120ms ease, box-shadow 120ms ease, background 180ms ease, color 120ms ease, border-color 120ms ease;
        backdrop-filter: saturate(110%) blur(2px);
    }

    .btn:hover {
        transform: translateY(-1px);
        box-shadow: 0 10px 16px -14px rgba(17,24,39,0.4);
        border-color: var(--op-ring);
    }

    .btn:active {
        transform: translateY(0);
    }

    .btn.primary {
        color: white;
        background:
            linear-gradient(180deg, color-mix(in oklab, var(--op-primary) 70%, #ffffff), color-mix(in oklab, var(--op-primary) 45%, #ffffff));
        border-color: color-mix(in oklab, var(--op-primary) 35%, #000);
        box-shadow: 0 14px 20px -12px rgba(37, 99, 235, 0.55);
    }

    .btn.primary:hover {
        box-shadow: 0 18px 26px -14px rgba(37, 99, 235, 0.6);
    }

    .btn.ghost {
        background: linear-gradient(180deg, rgba(37,99,235,0.08), rgba(37,99,235,0.04));
        color: var(--op-primary);
        border-color: color-mix(in oklab, var(--op-primary) 30%, var(--op-button-border));
    }

    /* Responsive tweaks */
    @media (max-width: 420px) {
        .status {
            flex-direction: column;
            align-items: stretch;
        }
        .turn {
            justify-content: space-between;
        }
    }
</style>
