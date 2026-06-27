import React, { useState, useEffect, useRef } from 'react';
import { 
  Send, MessageCircle, User, ShoppingBag, 
  Gamepad2, Database, Zap, Star, Users, Plus, X, Search, Mail, BatteryCharging
} from 'lucide-react';

const CHANNELS = ["Genel", "Oyun", "Tanışma", "Müzik", "Sanat", "Tarih", "Yazılım", "Spor", "Haberler", "Yardım"];

const MOCK_GLOBAL_USERS = [
  { id: 999, username: 'onur-kurucu', level: 99, avatar: '👑' },
  { id: 101, username: 'ProGamer99', level: 12, avatar: '🎮' },
  { id: 102, username: 'KediSever', level: 5, avatar: '🐱' },
  { id: 103, username: 'NexusMaster', level: 50, avatar: '💎' },
  { id: 104, username: 'GölgeSavaşçı', level: 25, avatar: '🗡️' },
];

const RARITIES = {
  Common: { name: 'Yaygın', color: 'text-slate-400', multiplier: 1, bg: 'bg-slate-800' },
  Rare: { name: 'Nadir', color: 'text-blue-400', multiplier: 2, bg: 'bg-blue-950' },
  Epic: { name: 'Epik', color: 'text-purple-500', multiplier: 4, bg: 'bg-purple-950' },
  Legendary: { name: 'Efsanevi', color: 'text-yellow-400', multiplier: 10, bg: 'bg-yellow-950' }
};

const SHOP_ITEMS = Array.from({ length: 80 }, (_, i) => {
  const types = ['frame', 'style'];
  const type = types[i % 2];
  const names = ["Obsidyen", "Kozmik", "Neon", "Gölge", "Işıltı", "Efsane", "Safir", "Zümrüt", "Kraliyet", "Süpernova"];
  const suffix = ["Çerçevesi", "Efekti", "Rozeti", "Parıltısı", "Hücresi"];
  
  return {
    id: `item_${i}`,
    name: `${names[i % names.length]} ${suffix[i % suffix.length]}`,
    cost: (Math.floor(i / 10) + 1) * 250, 
    type: type,
    value: type === 'frame' 
      ? `border-${['red', 'green', 'blue', 'purple', 'yellow', 'pink'][i % 6]}-500 border-2` 
      : `text-${['red', 'green', 'blue', 'purple', 'yellow', 'pink'][i % 6]}-400 font-bold`
  };
});

const ANIMALS = Array.from({ length: 50 }, (_, i) => ({
  id: i,
  name: `Yaratık #${i + 1}`,
  type: ['🦊', '🐻', '🐼', '🐨', '🐯', '🦁', '🐮', '🐷', '🐸', '🦄'][i % 10],
  rarity: i > 45 ? 'Legendary' : i > 35 ? 'Epic' : i > 15 ? 'Rare' : 'Common'
}));

export default function App() {
  const [view, setView] = useState('login');
  const [user, setUser] = useState({ username: '', coins: 1000, xp: 0, isAfk: false, energy: 100, avatar: '👤' });
  const [friends, setFriends] = useState([]);
  const [farm, setFarm] = useState([]);
  const [inventory, setInventory] = useState([]);
  const [activeFrame, setActiveFrame] = useState('border-slate-700 border-2');
  const [activeStyle, setActiveStyle] = useState('text-slate-200');
  const [messages, setMessages] = useState([]);
  const [inputText, setInputText] = useState('');
  const [searchQuery, setSearchQuery] = useState('');
  const [activeChannel, setActiveChannel] = useState('Genel');
  const [activeDmUser, setActiveDmUser] = useState(null);
  const chatEndRef = useRef(null);

  useEffect(() => {
    const interval = setInterval(() => {
      setUser(prev => ({ ...prev, energy: Math.min(prev.energy + 5, 100) }));
    }, 5000);
    return () => clearInterval(interval);
  }, []);

  useEffect(() => { chatEndRef.current?.scrollIntoView({ behavior: 'smooth' }); }, [messages, activeChannel, activeDmUser]);

  const handleSendMessage = () => {
    if (!inputText.trim()) return;
    const msg = { 
      id: Date.now(), 
      text: inputText, 
      user: user.username, 
      avatar: user.avatar,
      channel: activeDmUser ? null : activeChannel,
      targetUser: activeDmUser ? activeDmUser.username : null
    };
    setMessages([...messages, msg]);
    setUser(prev => ({ ...prev, xp: prev.xp + 10, coins: prev.coins + 1 }));
    setInputText('');
  };

  const handleHunt = () => {
    if (user.energy < 20) return;
    const rand = Math.random() * 100;
    let rarityKey = rand > 95 ? 'Legendary' : rand > 80 ? 'Epic' : rand > 50 ? 'Rare' : 'Common';
    setFarm(prev => [...prev, { ...ANIMALS[Math.floor(Math.random() * ANIMALS.length)], rarity: rarityKey, instanceId: Date.now() }]);
    setUser(prev => ({ ...prev, energy: prev.energy - 20 }));
  };

  const sellAnimal = (animal) => {
    const mult = RARITIES[animal.rarity].multiplier;
    setUser(prev => ({ ...prev, coins: prev.coins + Math.floor(100 * mult), xp: prev.xp + 50 }));
    setFarm(farm.filter(a => a.instanceId !== animal.instanceId));
  };

  if (view === 'login') return (
    <div className="flex h-screen items-center justify-center bg-slate-950">
      <div className="bg-slate-900 p-8 rounded-2xl border border-slate-800 text-center shadow-xl w-80">
        <h1 className="text-3xl font-bold mb-6 text-indigo-400">Social Nexus</h1>
        <input 
          placeholder="Kullanıcı adın..."
          className="bg-slate-800 p-3 rounded-lg w-full mb-4 text-center border border-slate-700 outline-none text-white"
          onKeyDown={(e) => { if(e.key === 'Enter' && e.target.value) { setUser({...user, username: e.target.value}); setView('chat'); } }}
        />
      </div>
    </div>
  );

  return (
    <div className="flex h-screen bg-slate-950 text-slate-200">
      <div className="w-64 bg-slate-900 border-r border-slate-800 p-4 flex flex-col">
        <h2 className="text-xl font-bold text-white mb-6 flex items-center gap-2"><Star className="text-yellow-400" /> Nexus</h2>
        <div className="mb-6 bg-slate-800 p-4 rounded-xl border border-slate-700">
          <div className="flex items-center gap-3">
            <div className="text-3xl bg-slate-700 w-12 h-12 flex items-center justify-center rounded-full">{user.avatar}</div>
            <div>
              <div className="font-bold text-white">{user.username}</div>
              <div className="text-xs text-slate-400">Level {Math.floor(user.xp / 500) + 1}</div>
            </div>
          </div>
          <div className="w-full bg-slate-700 h-2 rounded-full mt-2"><div className="h-full bg-indigo-500 rounded-full" style={{width: `${((user.xp % 500) / 500) * 100}%`}}></div></div>
          <div className="mt-3 flex justify-between text-sm">
            <span className="text-yellow-400 font-bold">{user.coins} 💰</span>
            <span className="text-blue-400 font-bold flex items-center gap-1"><BatteryCharging size={14}/> {user.energy}</span>
          </div>
        </div>

        <nav className="flex-1 space-y-2">
          {[
            { id: 'chat', label: 'Sohbet', icon: <MessageCircle /> },
            { id: 'farm', label: 'Çiftlik', icon: <Database /> },
            { id: 'shop', label: 'Mağaza', icon: <ShoppingBag /> },
            { id: 'friends', label: 'Arkadaşlar', icon: <Users /> },
            { id: 'profile', label: 'Profil', icon: <User /> }
          ].map(item => (
            <button key={item.id} onClick={() => { setView(item.id); setActiveDmUser(null); }} className={`w-full flex items-center gap-3 p-3 rounded-xl transition ${view === item.id ? 'bg-indigo-600 text-white' : 'hover:bg-slate-800 text-slate-400'}`}>
              {item.icon} {item.label}
            </button>
          ))}
        </nav>
      </div>

      <div className="flex-1 flex flex-col overflow-hidden bg-slate-950">
        
        {view === 'chat' && (
          <div className="flex h-full">
            <div className="w-48 border-r border-slate-800 p-2 overflow-y-auto">
              <h3 className="text-sm font-bold text-slate-500 mb-2 uppercase">Kanallar</h3>
              {CHANNELS.map(c => (
                <button key={c} onClick={() => { setActiveChannel(c); setActiveDmUser(null); }} className={`w-full text-left p-2 rounded-lg text-sm ${activeChannel === c && !activeDmUser ? 'bg-slate-800 text-indigo-400 font-bold' : 'text-slate-400'}`}># {c}</button>
              ))}
            </div>
            <div className="flex-1 flex flex-col">
              <div className="p-4 border-b border-slate-800 font-bold">{activeDmUser ? `DM: ${activeDmUser.username}` : `# ${activeChannel}`}</div>
              <div className="flex-1 overflow-y-auto p-6 space-y-4">
                {messages.filter(m => activeDmUser ? ((m.user === user.username && m.targetUser === activeDmUser.username) || (m.user === activeDmUser.username && m.targetUser === user.username)) : m.channel === activeChannel).map(m => (
                  <div key={m.id} className="flex gap-4">
                    <div className="w-8 h-8 rounded-full bg-slate-800 flex items-center justify-center shrink-0">{m.avatar}</div>
                    <div className={`p-3 rounded-xl bg-slate-800 max-w-lg ${activeFrame} ${activeStyle} shadow-lg`}>
                      <p className="text-xs text-slate-500 mb-1">{m.user}</p>
                      {m.text}
                    </div>
                  </div>
                ))}
                <div ref={chatEndRef} />
              </div>
              <div className="p-4 border-t border-slate-800 bg-slate-900">
                <input value={inputText} onChange={e => setInputText(e.target.value)} onKeyDown={e => e.key === 'Enter' && handleSendMessage()} className="w-full bg-slate-800 p-4 rounded-xl border border-slate-700 text-white outline-none" placeholder="Bir şeyler yaz..." />
              </div>
            </div>
          </div>
        )}

        {view === 'farm' && (
          <div className="p-8 overflow-y-auto">
             <div className="flex justify-between items-center mb-6">
              <h2 className="text-2xl font-bold">Çiftliğim</h2>
              <button onClick={handleHunt} disabled={user.energy < 20} className="bg-indigo-600 px-6 py-2 rounded-xl font-bold flex items-center gap-2">
                <Zap size={16}/> Avlan (20 Enerji)
              </button>
            </div>
            <div className="grid grid-cols-4 gap-4">
              {[...farm].sort((a,b) => RARITIES[b.rarity].multiplier - RARITIES[a.rarity].multiplier).map(a => (
                <div key={a.instanceId} className={`${RARITIES[a.rarity].bg} p-4 rounded-2xl border border-slate-700 text-center`}>
                  <div className="text-4xl mb-2">{a.type}</div>
                  <div className={`text-xs font-bold ${RARITIES[a.rarity].color}`}>{a.rarity}</div>
                  <button onClick={() => sellAnimal(a)} className="bg-black/20 w-full mt-3 py-1 rounded-lg text-sm">Sat</button>
                </div>
              ))}
            </div>
          </div>
        )}

        {view === 'shop' && (
          <div className="p-8 overflow-y-auto">
            <h2 className="text-2xl font-bold mb-6">Mağaza</h2>
            <div className="grid grid-cols-3 gap-4">
              {SHOP_ITEMS.map(item => (
                <div key={item.id} className="bg-slate-900 p-4 rounded-2xl border border-slate-800">
                  <h3 className="font-bold text-sm">{item.name}</h3>
                  <p className="text-yellow-400 text-sm">{item.cost} Coin</p>
                  <button onClick={() => { if(user.coins >= item.cost) { setUser({...user, coins: user.coins - item.cost}); setInventory([...inventory, item]); } }} className="w-full mt-4 bg-slate-800 py-2 rounded-lg text-sm">Satın Al</button>
                </div>
              ))}
            </div>
          </div>
        )}

        {view === 'friends' && (
          <div className="p-8 grid grid-cols-2 gap-8 h-full">
            <div className="bg-slate-900 p-6 rounded-2xl border border-slate-800">
              <h3 className="font-bold text-xl mb-4 flex items-center gap-2"><Users className="text-indigo-400"/> Arkadaşlarım</h3>
              {friends.map(f => (
                  <div key={f.id} className="bg-slate-800 p-3 rounded-lg flex justify-between items-center border border-slate-700 mb-2">
                    <span>{f.avatar} {f.username}</span>
                    <div className="flex gap-2">
                      <button onClick={() => { setView('chat'); setActiveDmUser(f); }} className="text-indigo-400"><Mail size={18}/></button>
                    </div>
                  </div>
                ))}
            </div>
            <div className="bg-slate-900 p-6 rounded-2xl border border-slate-800">
              <h3 className="font-bold text-xl mb-4 flex items-center gap-2"><Search className="text-emerald-400"/> Kullanıcıları Keşfet</h3>
              <input onChange={e => setSearchQuery(e.target.value)} placeholder="İsim ile ara..." className="w-full bg-slate-800 p-2 rounded-lg mb-4 border border-slate-700 outline-none"/>
              {MOCK_GLOBAL_USERS.filter(u => u.username.toLowerCase().includes(searchQuery.toLowerCase()) && !friends.find(f => f.id === u.id)).map(u => (
                  <div key={u.id} className="bg-slate-800 p-3 rounded-lg flex justify-between items-center border border-slate-700 mb-2">
                    <span>{u.avatar} {u.username}</span>
                    <button onClick={() => setFriends([...friends, u])} className="text-indigo-400"><Plus size={18}/></button>
                  </div>
                ))}
            </div>
          </div>
        )}

        {view === 'profile' && (
          <div className="p-8 overflow-y-auto">
            <h2 className="text-2xl font-bold mb-6">Profil Düzenle</h2>
            <div className="bg-slate-900 p-6 rounded-2xl border border-slate-800 mb-8">
              <h3 className="font-bold mb-4">Avatarını Seç</h3>
              <div className="grid grid-cols-8 gap-2">
                {['👤', '🦊', '🐻', '🐼', '🐨', '🐯', '🦁', '🐮', '🐷', '🐸', '🦄', '🤖', '👾', '🚀', '🌟', '🔥', '💎', '👑'].map(icon => (
                  <button key={icon} onClick={() => setUser({...user, avatar: icon})} className={`p-3 rounded-lg text-2xl ${user.avatar === icon ? 'bg-indigo-600' : 'bg-slate-800'}`}>
                    {icon}
                  </button>
                ))}
              </div>
            </div>
            <h2 className="text-xl font-bold mb-4">Envanterin</h2>
            <div className="grid grid-cols-3 gap-4">
              {inventory.map((item, i) => {
                const isEquipped = activeFrame === item.value || activeStyle === item.value;
                return (
                  <div key={i} className={`bg-slate-800 p-6 rounded-2xl border ${isEquipped ? 'border-green-500' : 'border-slate-700'}`}>
                    <p className="font-bold">{item.name}</p>
                    <button 
                      onClick={() => { if(item.type === 'frame') setActiveFrame(item.value); else setActiveStyle(item.value); }} 
                      className={`mt-4 w-full py-2 rounded-lg text-sm ${isEquipped ? 'bg-green-600' : 'bg-indigo-600'}`}
                    >
                      {isEquipped ? 'Donatıldı' : 'Donat'}
                    </button>
                  </div>
                );
              })}
            </div>
          </div>
        )}
      </div>
    </div>
  );
}
