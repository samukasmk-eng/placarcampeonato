import React, { useState } from "react";

// Placar Kasual - Single-file React component
// TailwindCSS classes are used for styling (no import required in the canvas preview).
// Default export is a React component ready to drop into a Next.js / Vite + React app.

// How it works:
// - mantém uma lista de times (trios) com pontuação, kills e colocações
// - ao atingir 200 pontos, o jogador/time entra em MATCH POINT (badge)
// - cálculo de pontos por kill/posição segue o sistema fornecido

const KILL_MULTIPLIERS = {
  1: 2.0,
  2: 1.8,
  3: 1.6,
  4: 1.4,
  5: 1.2,
};

function calcKillPoints(killPlaceCounts) {
  // killPlaceCounts: objeto {1: qtd, 2: qtd, ...}
  let pts = 0;
  for (const placeStr of Object.keys(killPlaceCounts)) {
    const place = Number(placeStr);
    const count = Number(killPlaceCounts[placeStr]) || 0;
    const mult = KILL_MULTIPLIERS[place] ?? 1.0;
    pts += count * mult;
  }
  return pts;
}

function sampleTeams() {
  return [
    { id: 1, name: "SMK Alpha", players: ["JogadorA", "JogadorB", "JogadorC"], score: 186, kills: 18 },
    { id: 2, name: "SMK Bravo", players: ["JogadorD", "JogadorE", "JogadorF"], score: 203, kills: 21 },
    { id: 3, name: "SMK Charlie", players: ["JogadorG", "JogadorH", "JogadorI"], score: 124, kills: 9 },
  ];
}

export default function PlacarKasual() {
  const [teams, setTeams] = useState(sampleTeams());
  const [newTeamName, setNewTeamName] = useState("");

  function addTeam() {
    if (!newTeamName.trim()) return;
    const id = Date.now();
    const t = { id, name: newTeamName.trim(), players: [], score: 0, kills: 0 };
    setTeams((s) => [...s, t]);
    setNewTeamName("");
  }

  function updateScore(id, delta) {
    setTeams((prev) => prev.map((t) => (t.id === id ? { ...t, score: Math.max(0, t.score + delta) } : t)));
  }

  function setKills(id, kills) {
    setTeams((prev) => prev.map((t) => (t.id === id ? { ...t, kills: Number(kills) } : t)));
  }

  function resetScores() {
    setTeams((prev) => prev.map((t) => ({ ...t, score: 0, kills: 0 })));
  }

  // Sort by score desc
  const sorted = [...teams].sort((a, b) => b.score - a.score || b.kills - a.kills);

  return (
    <div className="min-h-screen bg-gradient-to-b from-neutral-900 via-neutral-800 to-neutral-900 text-slate-100 p-6">
      <header className="max-w-6xl mx-auto flex items-center justify-between mb-6">
        <div>
          <h1 className="text-3xl font-extrabold">Campeonato Kasual — Painel</h1>
          <p className="text-sm text-slate-300">Placar dinâmico para torneios de Warzone — baseado em "Placar Kasual".</p>
        </div>
        <div className="flex gap-2">
          <button
            onClick={resetScores}
            className="bg-red-600/90 hover:bg-red-700 px-4 py-2 rounded-2xl shadow-sm text-sm"
          >
            Resetar Placar
          </button>
        </div>
      </header>

      <main className="max-w-6xl mx-auto grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Scoreboard */}
        <section className="lg:col-span-2 bg-neutral-800/50 p-4 rounded-2xl shadow-md">
          <h2 className="text-xl font-bold mb-3">Posições</h2>

          <div className="space-y-3">
            {sorted.map((team, idx) => (
              <article key={team.id} className="flex items-center justify-between p-3 rounded-xl bg-neutral-900/40">
                <div className="flex items-center gap-3">
                  <div className="w-12 h-12 rounded-full bg-gradient-to-br from-indigo-600 to-pink-600 flex items-center justify-center font-bold">{idx + 1}</div>
                  <div>
                    <div className="font-semibold text-lg">{team.name}</div>
                    <div className="text-xs text-slate-400">{team.players.join(' • ') || 'Sem jogadores cadastrados'}</div>
                  </div>
                </div>

                <div className="flex items-center gap-4">
                  <div className="text-right">
                    <div className="text-2xl font-extrabold">{team.score}</div>
                    <div className="text-xs text-slate-400">Kills: {team.kills}</div>
                  </div>

                  {/* Match Point badge */}
                  {team.score >= 200 && (
                    <div className="px-3 py-1 rounded-full bg-yellow-400 text-black font-semibold text-sm">MATCH POINT</div>
                  )}

                  {/* Controls */}
                  <div className="flex items-center gap-2">
                    <button onClick={() => updateScore(team.id, 5)} className="px-3 py-1 rounded bg-green-600/90 text-sm">+5</button>
                    <button onClick={() => updateScore(team.id, -5)} className="px-3 py-1 rounded bg-red-600/90 text-sm">-5</button>
                  </div>
                </div>
              </article>
            ))}
          </div>
        </section>

        {/* Sidebar: Adicionar time, cálculo de kills, regras */}
        <aside className="bg-neutral-800/50 p-4 rounded-2xl shadow-md">
          <h3 className="font-bold mb-2">Adicionar / editar</h3>

          <div className="mb-3">
            <input value={newTeamName} onChange={(e) => setNewTeamName(e.target.value)} placeholder="Nome do trio" className="w-full rounded px-3 py-2 bg-neutral-900/60" />
            <div className="flex gap-2 mt-2">
              <button onClick={addTeam} className="flex-1 py-2 rounded bg-indigo-600/90">Adicionar Time</button>
            </div>
          </div>

          <div className="mb-3">
            <h4 className="font-semibold">Calculadora de Kills</h4>
            <p className="text-xs text-slate-400 mb-2">Insira quantas kills em cada categoria (1º, 2º, 3º...) para ver os pontos gerados.</p>
            <KillCalculator />
          </div>

          <div>
            <h4 className="font-semibold">Regras Rápidas</h4>
            <ul className="text-sm text-slate-300 ml-4 list-disc mt-2">
              <li>Quando um time atingir <span className="font-bold">200 pontos</span>, ele entra em <span className="font-bold">MATCH POINT</span>.</li>
              <li>Kills pontuam segundo multiplicadores (1º kill x2.0, 2º x1.8, 3º x1.6, 4º x1.4, 5º x1.2).</li>
              <li>Atualize o placar manualmente ou conecte ao backend para atualizações em tempo real.</li>
            </ul>
          </div>
        </aside>
      </main>

      <footer className="max-w-6xl mx-auto mt-8 text-center text-slate-400 text-sm">
        Built for Campeonato Kasual — adaptação do site-base fornecido.
      </footer>
    </div>
  );
}

function KillCalculator() {
  const [counts, setCounts] = useState({ 1: 0, 2: 0, 3: 0, 4: 0, 5: 0 });

  function change(place, v) {
    setCounts((p) => ({ ...p, [place]: Math.max(0, Number(v || 0)) }));
  }

  const total = calcKillPoints(counts);

  return (
    <div className="bg-neutral-900/40 p-3 rounded">
      <div className="grid grid-cols-2 gap-2 mb-2">
        {[1, 2, 3, 4, 5].map((p) => (
          <label key={p} className="text-xs">
            {p}º x{KILL_MULTIPLIERS[p]}
            <input type="number" min={0} value={counts[p]} onChange={(e) => change(p, e.target.value)} className="w-full mt-1 rounded px-2 py-1 bg-neutral-800/50" />
          </label>
        ))}
      </div>
      <div className="text-sm font-semibold">Pontos por kills: {total}</div>
    </div>
  );
}
