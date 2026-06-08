# CODSOFT-Rock-Paper-Scissors-Game
Excited to share that I have successfully built a Rock-Paper-Scissors Game as part of my programming learning journey. #Python #Programming #Coding #GameDevelopment #PythonProjects #LearningByDoing #DeveloperJourney #TechSkills #ComputerScience #100DaysOfCode #CODSOFT
Author- Prem Somnath Sonawane
https://ai.studio/apps/b34e961e-381f-4c68-947b-e0c695af00d6?fullscreenApplet=true

import { useState, useEffect } from 'react';
import { Choice, Difficulty, GameStats, HistoryItem } from './types';
import { getComputerChoice, getResult, playSound } from './utils/gameHelpers';
import DifficultySelector from './components/DifficultySelector';
import StatsDashboard from './components/StatsDashboard';
import PlayArea from './components/PlayArea';
import HistoryLog from './components/HistoryLog';
import { Volume2, VolumeX, Sun, Moon, HelpCircle, X } from 'lucide-react';

const LOCAL_STORAGE_STATS_KEY = 'rps_game_stats_v2';
const LOCAL_STORAGE_HISTORY_KEY = 'rps_game_history_v2';
const LOCAL_STORAGE_DIFFICULTY_KEY = 'rps_game_diff_v2';
const LOCAL_STORAGE_SOUND_KEY = 'rps_game_sound_v2';
const LOCAL_STORAGE_THEME_KEY = 'rps_game_theme_v2';

export default function App() {
  const [stats, setStats] = useState<GameStats>(() => {
    try {
      const stored = localStorage.getItem(LOCAL_STORAGE_STATS_KEY);
      return stored
        ? JSON.parse(stored)
        : { wins: 0, losses: 0, ties: 0, currentStreak: 0, bestStreak: 0 };
    } catch {
      return { wins: 0, losses: 0, ties: 0, currentStreak: 0, bestStreak: 0 };
    }
  });

  const [history, setHistory] = useState<HistoryItem[]>(() => {
    try {
      const stored = localStorage.getItem(LOCAL_STORAGE_HISTORY_KEY);
      return stored ? JSON.parse(stored) : [];
    } catch {
      return [];
    }
  });

  const [difficulty, setDifficulty] = useState<Difficulty>(() => {
    try {
      const stored = localStorage.getItem(LOCAL_STORAGE_DIFFICULTY_KEY);
      return (stored as Difficulty) || 'medium';
    } catch {
      return 'medium';
    }
  });

  const [soundEnabled, setSoundEnabled] = useState<boolean>(() => {
    try {
      const stored = localStorage.getItem(LOCAL_STORAGE_SOUND_KEY);
      return stored !== 'false';
    } catch {
      return true;
    }
  });

  const [darkMode, setDarkMode] = useState<boolean>(() => {
    try {
      const stored = localStorage.getItem(LOCAL_STORAGE_THEME_KEY);
      if (stored) return stored === 'true';
      return true; // Default to dark theme for Sleek Interface
    } catch {
      return true;
    }
  });

  const [showHowToPlay, setShowHowToPlay] = useState<boolean>(false);

  useEffect(() => {
    localStorage.setItem(LOCAL_STORAGE_STATS_KEY, JSON.stringify(stats));
  }, [stats]);

  useEffect(() => {
    localStorage.setItem(LOCAL_STORAGE_HISTORY_KEY, JSON.stringify(history));
  }, [history]);

  useEffect(() => {
    localStorage.setItem(LOCAL_STORAGE_DIFFICULTY_KEY, difficulty);
  }, [difficulty]);

  useEffect(() => {
    localStorage.setItem(LOCAL_STORAGE_SOUND_KEY, String(soundEnabled));
  }, [soundEnabled]);

  useEffect(() => {
    localStorage.setItem(LOCAL_STORAGE_THEME_KEY, String(darkMode));
    if (darkMode) {
      document.documentElement.classList.add('dark');
    } else {
      document.documentElement.classList.remove('dark');
    }
  }, [darkMode]);

  const handleDifficultyChange = (newDiff: Difficulty) => {
    setDifficulty(newDiff);
    if (soundEnabled) {
      playSound('select');
    }
  };

  const handlePlayRound = (user: Choice, computer: Choice) => {
    const outcome = getResult(user, computer);

    setStats((prev) => {
      let newWins = prev.wins;
      let newLosses = prev.losses;
      let newTies = prev.ties;
      let newStreak = prev.currentStreak;

      if (outcome === 'win') {
        newWins += 1;
        newStreak += 1;
      } else if (outcome === 'lose') {
        newLosses += 1;
        newStreak = 0;
      } else {
        newTies += 1;
      }

      const newBestStreak = Math.max(prev.bestStreak, newStreak);

      return {
        wins: newWins,
        losses: newLosses,
        ties: newTies,
        currentStreak: newStreak,
        bestStreak: newBestStreak
      };
    });

    setHistory((prev) => {
      const nextRoundNumber = prev.length > 0 ? prev[0].roundNumber + 1 : 1;
      const newItem: HistoryItem = {
        id: crypto.randomUUID(),
        roundNumber: nextRoundNumber,
        userChoice: user,
        computerChoice: computer,
        result: outcome,
        difficulty,
        timestamp: new Date().toISOString()
      };
      return [newItem, ...prev].slice(0, 15);
    });
  };

  const handleResetAll = () => {
    const confirmReset = window.confirm('Are you absolutely sure you want to reset all user scores, streaking counts, and battle chronology logs?');
    if (confirmReset) {
      setStats({ wins: 0, losses: 0, ties: 0, currentStreak: 0, bestStreak: 0 });
      setHistory([]);
      if (soundEnabled) {
        playSound('select');
      }
    }
  };

  const totalGames = stats.wins + stats.losses + stats.ties;

  return (
    <div className="min-h-screen bg-slate-950 dark:bg-slate-950 text-slate-100 flex flex-col md:flex-row overflow-y-auto md:overflow-hidden font-sans select-none">
      {/* Sidebar Aside Panel */}
      <aside className="w-full md:w-80 bg-slate-900 border-b md:border-b-0 md:border-r border-slate-800 flex flex-col p-6 shrink-0">
        {/* Brand Header */}
        <div className="flex items-center gap-3 mb-8">
          <div className="w-10 h-10 rounded-xl bg-indigo-600 flex items-center justify-center shadow-lg shadow-indigo-500/20">
            <span className="text-xl font-black italic text-white font-display">R</span>
          </div>
          <div>
            <h1 className="text-lg font-bold tracking-tight bg-gradient-to-r from-white to-slate-400 bg-clip-text text-transparent">
              RPS PRO
            </h1>
            <span className="text-[10px] font-mono font-semibold text-slate-500 tracking-wider block uppercase leading-none mt-0.5">
              Tactical Match Arena
            </span>
          </div>
        </div>

        {/* Tactical Overview */}
        <div className="space-y-4 mb-6">
          <span className="text-[9px] font-mono text-slate-500 font-bold uppercase tracking-[0.2em] block">
            SESSION OVERVIEW
          </span>
          <div className="grid grid-cols-2 gap-3.5">
            <div className="p-3.5 rounded-xl bg-slate-950/60 border border-slate-800/80">
              <span className="block text-[10px] text-slate-500 font-bold uppercase tracking-wider mb-1">WIN RATE</span>
              <span className="text-xl font-black text-emerald-400 font-display">
                {totalGames > 0 ? Math.round((stats.wins / totalGames) * 100) : 0}%
              </span>
            </div>
            <div className="p-3.5 rounded-xl bg-slate-950/60 border border-slate-800/80">
              <span className="block text-[10px] text-slate-500 font-bold uppercase tracking-wider mb-1">STREAK</span>
              <span className="text-xl font-black text-orange-400 font-display">
                {stats.currentStreak} 🔥
              </span>
            </div>
          </div>
        </div>

        {/* Sidebar History List */}
        <div className="flex-1 flex flex-col min-h-0 space-y-3 justify-between">
          <HistoryLog history={history} onResetAll={handleResetAll} />
        </div>
      </aside>

      {/* Main Panel Stage Right */}
      <main className="flex-1 flex flex-col p-6 md:p-8 relative min-h-0 overflow-y-auto">
        {/* Header Scores Row */}
        <div className="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-4 mb-8">
          {/* Real-time scoreboard readout */}
          <div className="flex items-center gap-6">
            <div>
              <div className="text-[9px] font-mono text-slate-500 tracking-[0.15em] font-bold uppercase">PLAYER SCORE</div>
              <div className="text-3xl font-black text-white font-display mt-0.5">{stats.wins}</div>
            </div>
            <div className="w-px h-10 bg-slate-800" />
            <div>
              <div className="text-[9px] font-mono text-slate-500 tracking-[0.15em] font-bold uppercase">AI SCORE</div>
              <div className="text-3xl font-black text-white font-display mt-0.5">{stats.losses}</div>
            </div>
          </div>

          {/* Quick Controls Layout */}
          <div className="flex items-center gap-3">
            {/* System Signal Online Badge */}
            <div className="flex items-center gap-2 px-3 py-1.5 rounded-lg bg-emerald-500/5 border border-emerald-500/20 mr-2">
              <span className="w-2 h-2 rounded-full bg-emerald-450 animate-pulse" />
              <span className="text-[9px] font-mono font-bold text-emerald-400 uppercase tracking-widest">
                SYSTEM ONLINE
              </span>
            </div>

            <button
              onClick={() => setShowHowToPlay(true)}
              className="p-2 text-slate-400 hover:text-white rounded-lg hover:bg-slate-900 border border-slate-800 transition cursor-pointer"
              title="Match Instructions"
            >
              <HelpCircle size={16} />
            </button>
            <button
              onClick={() => {
                setSoundEnabled(!soundEnabled);
                if (!soundEnabled) {
                  setTimeout(() => playSound('select'), 50);
                }
              }}
              className="p-2 text-slate-400 hover:text-white rounded-lg hover:bg-slate-900 border border-slate-800 transition cursor-pointer"
              title={soundEnabled ? 'Mute Sounds' : 'Unmute Sounds'}
            >
              {soundEnabled ? <Volume2 size={16} /> : <VolumeX size={16} />}
            </button>
            <button
              onClick={() => {
                setDarkMode(!darkMode);
                if (soundEnabled) playSound('select');
              }}
              className="p-2 text-slate-400 hover:text-white rounded-lg hover:bg-slate-900 border border-slate-800 transition cursor-pointer"
              title={darkMode ? 'Light Space' : 'Dark Space'}
            >
              {darkMode ? <Sun size={16} /> : <Moon size={16} />}
            </button>
          </div>
        </div>

        {/* Core Battleground Duel Card */}
        <div className="flex-1 flex flex-col justify-center max-w-2xl mx-auto w-full gap-8">
          {/* Active Battle Arena Component */}
          <PlayArea
            onPlayRound={handlePlayRound}
            onGetComputerChoice={(userMove) => getComputerChoice(userMove, difficulty, history)}
            soundEnabled={soundEnabled}
          />

          {/* Cognitive Difficulty Settings Selection Row */}
          <div className="p-5 rounded-2xl bg-slate-900 border border-slate-800 shadow-sm mt-2">
            <DifficultySelector
              currentDifficulty={difficulty}
              onChange={handleDifficultyChange}
              disabled={history.length > 0 && !history[0].userChoice}
            />
          </div>

          {/* Gameplay Ratio Distributed Meter */}
          <div className="p-5 rounded-2xl bg-slate-900 border border-slate-800 shadow-sm">
            <StatsDashboard stats={stats} />
          </div>
        </div>

        {/* Immersive Sandbox Console Metadata Footer */}
        <footer className="mt-8 border-t border-slate-900 pt-5 flex flex-col sm:flex-row justify-between items-center text-[10px] font-mono text-slate-500 tracking-wider gap-2">
          <div>
            COGNITION LEVEL: <span className="text-slate-300 font-bold">{difficulty.toUpperCase()}</span> • TOTAL TRIALS: <span className="text-slate-300 font-bold">{totalGames}</span>
          </div>
          <div>
            SANDBOX SERVER: <span className="text-indigo-400 font-bold">LOCAL-NODE-SECURE</span>
          </div>
        </footer>
      </main>

      {/* Modern High-contrasted Info Overlay Modals */}
      {showHowToPlay && (
        <div className="fixed inset-0 bg-slate-950/85 backdrop-blur-sm z-50 flex items-center justify-center p-4">
          <div className="bg-slate-900 border border-slate-800 max-w-md w-full rounded-2xl p-6 shadow-2xl relative animate-in fade-in zoom-in-95 duration-150 text-slate-100">
            <button
              onClick={() => {
                setShowHowToPlay(false);
                if (soundEnabled) playSound('select');
              }}
              className="absolute top-4 right-4 text-slate-500 hover:text-white cursor-pointer"
            >
              <X size={18} />
            </button>
            <h3 className="font-display font-bold text-lg text-white mb-2.5 flex items-center gap-2">
              <span>⚔️</span> Arena Battle Rules
            </h3>
            <div className="text-xs text-slate-400 flex flex-col gap-3.5 leading-relaxed font-sans">
              <p>
                Rock-Paper-Scissors is a universal match of dynamic decisions. The rules are absolute:
              </p>
              <ul className="list-disc pl-5 font-mono text-xs text-slate-300 flex flex-col gap-1.5 bg-slate-950/40 p-3 rounded-lg border border-slate-800/60">
                <li>🪨 Rock beats ✂️ Scissors</li>
                <li>✂️ Scissors beats 📄 Paper</li>
                <li>📄 Paper beats 🪨 Rock</li>
              </ul>
              <h4 className="font-semibold text-slate-200 mt-2">
                COGNITIVE PREDICTIVE MODELS:
              </h4>
              <p>
                <strong>Easy</strong> drops algorithmic accuracy to grant you frequent wins.
                <br />
                <strong>Medium</strong> relies on standard 100% fair random distribution.
                <br />
                <strong>Hard</strong> employs smart pattern-decoding algorithms (Markov Chains & psychological Win-Stay-Lose-Shift theory) matching your previous moves to predict your reactions. Beat the logic if you can!
              </p>
            </div>
          </div>
        </div>
      )}
    </div>
  );
}
