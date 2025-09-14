E-Sports Tournament Web App (Professional Look — React + Tailwind + shadcn/ui + Framer Motion)

import React, { useEffect, useState } from 'react';
import { motion } from 'framer-motion';
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { saveAs } from 'file-saver';

// Local storage hook
const useLocalState = (key, initial) => {
  const [state, setState] = useState(() => {
    try { const s = localStorage.getItem(key); return s ? JSON.parse(s) : initial; } catch { return initial; }
  });
  useEffect(() => { localStorage.setItem(key, JSON.stringify(state)); }, [key, state]);
  return [state, setState];
};

export default function TournamentApp(){
  const [players, setPlayers] = useLocalState('players', []);
  const [teams, setTeams] = useLocalState('teams', []);
  const [matches, setMatches] = useLocalState('matches', []);
  const [announcements, setAnnouncements] = useLocalState('announcements', []);
  const [adminMode, setAdminMode] = useState(false);
  const ADMIN_PASS = 'admin123';

  const [form, setForm] = useState({name:'', team:'', game:''});

  const addPlayer = (e)=>{
    e.preventDefault();
    if(!form.name || !form.team || !form.game) return;
    if(!teams.find(t=>t.name===form.team)){
      setTeams([...teams, {name: form.team, game: form.game}]);
    }
    setPlayers([...players, {...form, id: Date.now()}]);
    setForm({name:'', team:'', game:''});
  };

  const addMatch = (m)=> setMatches([...matches, {...m, id: Date.now()}]);
  const recordResult = (id, result)=> setMatches(matches.map(m=> m.id===id? {...m, result}:m));

  const leaderboard = (()=>{
    const map = {};
    teams.forEach(t=> map[t.name] = {team: t.name, played:0, won:0, lost:0, points:0});
    matches.forEach(m=>{
      if(!m.result) return;
      const A = map[m.teamA]; const B = map[m.teamB];
      if(!A||!B) return;
      A.played++; B.played++;
      if(m.result.winner===m.teamA){ A.won++; B.lost++; A.points+=3; }
      else if(m.result.winner===m.teamB){ B.won++; A.lost++; B.points+=3; }
      else { A.points++; B.points++; }
    });
    return Object.values(map).sort((a,b)=> b.points - a.points);
  })();

  return (
    <div className="min-h-screen bg-gradient-to-br from-gray-900 via-gray-800 to-black text-white p-6">
      <motion.header initial={{y:-40,opacity:0}} animate={{y:0,opacity:1}} className="flex justify-between items-center mb-6">
        <h1 className="text-3xl font-bold tracking-wide">🎮 E-Sports Tournament</h1>
        <div className="space-x-2">
          <Button onClick={()=>{const pass=prompt('Enter Admin Password'); if(pass===ADMIN_PASS) setAdminMode(true); else alert('Wrong password')}}>Admin</Button>
          <Button variant="destructive" onClick={()=>{setPlayers([]);setTeams([]);setMatches([]);setAnnouncements([]);localStorage.clear();}}>Reset</Button>
        </div>
      </motion.header>

      <main className="grid gap-6 md:grid-cols-3">
        {/* Registration */}
        <Card className="bg-gray-800 border-gray-700">
          <CardContent>
            <h2 className="text-xl font-semibold mb-4">Register Player</h2>
            <form onSubmit={addPlayer} className="space-y-3">
              <Input className="bg-gray-700 border-none" placeholder="Player Name" value={form.name} onChange={e=>setForm({...form,name:e.target.value})} />
              <Input className="bg-gray-700 border-none" placeholder="Team Name" value={form.team} onChange={e=>setForm({...form,team:e.target.value})} />
              <Input className="bg-gray-700 border-none" placeholder="Game (Free Fire, BGMI, Valorant)" value={form.game} onChange={e=>setForm({...form,game:e.target.value})} />
              <Button type="submit" className="w-full">Register</Button>
            </form>

            <h3 className="mt-6 font-medium">Players</h3>
            <div className="max-h-40 overflow-auto mt-2 space-y-2">
              {players.map(p=>(
                <motion.div key={p.id} initial={{opacity:0}} animate={{opacity:1}} className="p-2 bg-gray-700 rounded">
                  {p.name} — {p.team} ({p.game})
                </motion.div>
              ))}
            </div>
          </CardContent>
        </Card>

        {/* Matches */}
        <Card className="bg-gray-800 border-gray-700">
          <CardContent>
            <h2 className="text-xl font-semibold mb-4">Matches</h2>
            {adminMode && <MatchCreator teams={teams} onCreate={addMatch} />}
            <div className="space-y-3 max-h-72 overflow-auto">
              {matches.map(m=>(
                <motion.div key={m.id} initial={{y:20,opacity:0}} animate={{y:0,opacity:1}} className="p-3 bg-gray-700 rounded">
                  <div className="font-medium">{m.teamA} vs {m.teamB}</div>
                  <div className="text-sm">Date: {m.date}</div>
                  {m.result? (
                    <div className="text-sm text-green-400">Winner: {m.result.winner} ({m.result.scoreA}-{m.result.scoreB})</div>
                  ) : adminMode? <RecordResult match={m} onSave={recordResult} /> : <span className="text-sm">Pending</span>}
                </motion.div>
              ))}
            </div>
          </CardContent>
        </Card>

        {/* Leaderboard & Announcements */}
        <Card className="bg-gray-800 border-gray-700">
          <CardContent>
            <h2 className="text-xl font-semibold mb-4">Leaderboard</h2>
            <ul className="space-y-2">
              {leaderboard.map((l,i)=>(
                <li key={i} className="flex justify-between p-2 bg-gray-700 rounded">
                  <span>{i+1}. {l.team}</span>
                  <span className="font-bold">{l.points} pts</span>
                </li>
              ))}
            </ul>

            <h2 className="text-xl font-semibold mt-6 mb-2">Announcements</h2>
            <ul className="space-y-2">
              {announcements.map((a,i)=>(<li key={i} className="p-2 bg-gray-700 rounded">{a}</li>))}
            </ul>
            {adminMode && <AnnouncementEditor announcements={announcements} setAnnouncements={setAnnouncements} />}
          </CardContent>
        </Card>
      </main>
    </div>
  );
}

function MatchCreator({teams,onCreate}){
  const [tA,setTA]=useState(''); const [tB,setTB]=useState(''); const [date,setDate]=useState('');
  const create=()=>{if(!tA||!tB||tA===tB) return; onCreate({teamA:tA,teamB:tB,date}); setDate('');};
  return(
    <div className="space-y-2 mb-4">
      <select className="w-full bg-gray-700 p-2 rounded" value={tA} onChange={e=>setTA(e.target.value)}>
        <option value="">Select Team A</option>
        {teams.map(t=><option key={t.name}>{t.name}</option>)}
      </select>
      <select className="w-full bg-gray-700 p-2 rounded" value={tB} onChange={e=>setTB(e.target.value)}>
        <option value="">Select Team B</option>
        {teams.map(t=><option key={t.name}>{t.name}</option>)}
      </select>
      <input type="datetime-local" className="w-full bg-gray-700 p-2 rounded" value={date} onChange={e=>setDate(e.target.value)} />
      <Button onClick={create} className="w-full">Add Match</Button>
    </div>
  );
}

function RecordResult({match,onSave}){
  const [scoreA,setScoreA]=useState(0); const [scoreB,setScoreB]=useState(0);
  return(
    <div className="mt-2 space-y-2">
      <input type="number" className="w-full bg-gray-600 p-1 rounded" placeholder={`${match.teamA} Score`} value={scoreA} onChange={e=>setScoreA(+e.target.value)} />
      <input type="number" className="w-full bg-gray-600 p-1 rounded" placeholder={`${match.teamB} Score`} value={scoreB} onChange={e=>setScoreB(+e.target.value)} />
      <Button className="w-full" onClick={()=>onSave(match.id,{winner:scoreA===scoreB?"Draw":scoreA>scoreB?match.teamA:match.teamB,scoreA,scoreB})}>Save Result</Button>
    </div>
  );
}

function AnnouncementEditor({announcements,setAnnouncements}){
  const [text,setText]=useState('');
  const add=()=>{if(!text) return; setAnnouncements([text,...announcements]); setText('');};
  return(
    <div className="mt-3 space-y-2">
      <Input className="bg-gray-700 border-none" placeholder="Write announcement..." value={text} onChange={e=>setText(e.target.value)} />
      <Button onClick={add} className="w-full">Post</Button>
    </div>
  );
}


---

🔥 Changes for Professional Look

Dark gaming-style gradient background 🎮

Glass-like cards with Tailwind styling

Framer Motion animations (fade/slide)

Clean typography & spacing

Shadcn/ui buttons, inputs, cards



---

Bhai, ye ab ekdum gaming tournament dashboard jaisa lagega ✨
Tu chahe to main isme logos/icons + bracket view (knockout) bhi add kar du. Chahata hai?

