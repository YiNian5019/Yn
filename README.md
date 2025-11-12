import React, { useEffect, useState } from "react";

// Single-file React component Pokédex that queries PokéAPI
// Tailwind CSS utility classes assumed to be available in the host app.

export default function Pokedex() {
  const [query, setQuery] = useState(1); // default: Bulbasaur
  const [poke, setPoke] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetchPokemon(query);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  async function fetchPokemon(identifier) {
    if (!identifier && identifier !== 0) return;
    setLoading(true);
    setError(null);
    setPoke(null);
    const idOrName = String(identifier).trim().toLowerCase();
    try {
      const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${encodeURIComponent(idOrName)}`);
      if (!res.ok) throw new Error(`找不到寶可夢：${idOrName}`);
      const data = await res.json();

      // map the fields we care about
      const mapped = {
        id: data.id,
        name: data.name,
        sprites: {
          official: data.sprites.other?.['official-artwork']?.front_default || data.sprites.front_default,
          front: data.sprites.front_default,
          back: data.sprites.back_default,
        },
        types: data.types.map(t => t.type.name),
        stats: data.stats.map(s => ({ name: s.stat.name, base: s.base_stat })),
        height: data.height,
        weight: data.weight,
        abilities: data.abilities.map(a => ({ name: a.ability.name, hidden: a.is_hidden })),
      };
      setPoke(mapped);
    } catch (err) {
      setError(err.message || '發生錯誤');
    } finally {
      setLoading(false);
    }
  }

  function handleSearch(e) {
    e?.preventDefault();
    if (!query) return;
    fetchPokemon(query);
  }

  function handleRandom() {
    const randomId = Math.floor(Math.random() * 1010) + 1; // covers many generations
    setQuery(randomId);
    fetchPokemon(randomId);
  }

  function statBarWidth(base) {
    // simple normalization for display (max base stat often <=255)
    const max = 255;
    return Math.min(100, Math.round((base / max) * 100));
  }

  return (
    <div className="max-w-3xl mx-auto p-6">
      <h1 className="text-3xl font-bold mb-4">Pokédex (PokéAPI)</h1>

      <form onSubmit={handleSearch} className="flex gap-2 mb-4">
        <input
          aria-label="寶可夢名稱或編號"
          value={query}
          onChange={(e) => setQuery(e.target.value)}
          className="flex-1 p-2 border rounded shadow-sm"
          placeholder="輸入名稱（例：pikachu）或編號（例：25）"
        />
        <button type="submit" className="px-4 py-2 rounded bg-blue-600 text-white">查詢</button>
        <button type="button" onClick={handleRandom} className="px-4 py-2 rounded bg-gray-200">隨機</button>
      </form>

      {loading && (
        <div className="p-4">載入中…</div>
      )}

      {error && (
        <div className="p-4 bg-red-100 text-red-800 rounded">錯誤：{error}</div>
      )}

      {poke && (
        <div className="grid grid-cols-1 md:grid-cols-3 gap-6 items-start">
          <div className="col-span-1 flex flex-col items-center p-4 bg-white rounded shadow">
            <div className="text-sm text-gray-500">#{poke.id}</div>
            {poke.sprites.official ? (
              <img src={poke.sprites.official} alt={poke.name} className="w-48 h-48 object-contain" />
            ) : (
              <div className="w-48 h-48 flex items-center justify-center bg-gray-50">無圖片</div>
            )}
            <h2 className="capitalize text-2xl font-semibold mt-2">{poke.name}</h2>

            <div className="mt-3 flex gap-2">
              {poke.types.map(t => (
                <span key={t} className="capitalize px-3 py-1 rounded-full bg-gray-100 text-sm border">{t}</span>
              ))}
            </div>

            <div className="mt-3 text-sm text-gray-600">
              <div>身高: {poke.height}（dm）</div>
              <div>體重: {poke.weight}（hg）</div>
            </div>

            <div className="mt-3 w-full">
              <div className="text-sm font-medium">能力（abilities）</div>
              <ul className="text-sm mt-1 space-y-1">
                {poke.abilities.map(a => (
                  <li key={a.name} className="capitalize">{a.name}{a.hidden ? ' (隱藏)' : ''}</li>
                ))}
              </ul>
            </div>
          </div>

          <div className="col-span-2 p-4 bg-white rounded shadow">
            <h3 className="text-xl font-semibold mb-3">基礎能力（Base stats）</h3>
            <div className="space-y-3">
              {poke.stats.map(s => (
                <div key={s.name} className="">
                  <div className="flex justify-between text-sm mb-1">
                    <div className="capitalize font-medium">{s.name}</div>
                    <div className="font-mono">{s.base}</div>
                  </div>
                  <div className="w-full bg-gray-100 rounded h-3 overflow-hidden">
                    <div style={{ width: `${statBarWidth(s.base)}%` }} className="h-3 rounded bg-gradient-to-r from-green-400 to-yellow-400"></div>
                  </div>
                </div>
              ))}
            </div>

            <div className="mt-6">
              <h4 className="font-semibold mb-2">原始 JSON（調試用，可隱藏）</h4>
              <pre className="text-xs bg-gray-50 p-3 rounded max-h-72 overflow-auto">{JSON.stringify(poke, null, 2)}</pre>
            </div>
          </div>
        </div>
      )}

      <div className="mt-6 text-sm text-gray-500">
        注意：資料來源為 <a href="https://pokeapi.co/" className="underline">PokéAPI</a>。
      </div>
    </div>
  );
}
