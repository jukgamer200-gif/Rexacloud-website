import React, { useEffect, useState } from 'react';
import { motion, AnimatePresence } from 'motion/react';
import { 
  Server, 
  Shield, 
  Zap, 
  Cpu, 
  Globe, 
  Headphones, 
  ChevronRight, 
  Check,
  Menu,
  X,
  MessageSquare,
  Plus,
  Trash2,
  Edit2,
  LogOut,
  User as UserIcon,
  Settings,
  ArrowRight
} from 'lucide-react';
import { cn } from '@/src/lib/utils';
import { 
  collection, 
  onSnapshot, 
  addDoc, 
  deleteDoc, 
  doc, 
  updateDoc, 
  query, 
  orderBy,
  getDocFromServer,
  setDoc,
  getDoc
} from 'firebase/firestore';
import { 
  signInWithPopup, 
  GoogleAuthProvider, 
  onAuthStateChanged, 
  signOut,
  User
} from 'firebase/auth';
import { db, auth } from './firebase';

const DISCORD_LINK = "https://discord.gg/HZGBswA85Z";
const PANEL_LINK = "https://panel.rexacloud.fun/";

interface BannerSettings {
  text: string;
  code: string;
  discount: string;
  active: boolean;
}

const Banner = ({ settings }: { settings: BannerSettings | null }) => {
  const [visible, setVisible] = useState(true);
  const [copied, setCopied] = useState(false);
  
  if (!settings?.active) return null;

  const handleCopy = () => {
    navigator.clipboard.writeText(settings.code);
    setCopied(true);
    setTimeout(() => setCopied(false), 2000);
  };

  return (
    <AnimatePresence>
      {visible && (
        <motion.div 
          initial={{ y: -48, opacity: 0 }}
          animate={{ y: 0, opacity: 1 }}
          exit={{ y: -48, opacity: 0 }}
          transition={{ duration: 0.4, ease: [0.16, 1, 0.3, 1] }}
          className="relative z-[100] bg-brand-primary h-12 flex items-center justify-center overflow-hidden"
        >
          <div className="absolute inset-0 bg-[url('https://www.transparenttextures.com/patterns/carbon-fibre.png')] opacity-20" />
          <div className="max-w-7xl mx-auto px-4 w-full flex items-center justify-between relative">
            <div className="flex items-center gap-6">
              <span className="hidden md:block text-[9px] font-black uppercase tracking-[0.4em] bg-white/20 px-3 py-1 rounded text-white">Limited Offer</span>
              <p className="text-[10px] sm:text-[12px] font-black uppercase tracking-[0.1em] text-white">
                {settings.text} — <span className="opacity-80">SAVE {settings.discount} TODAY</span>
              </p>
            </div>

            <div className="flex items-center gap-6">
              <button 
                onClick={handleCopy}
                className="flex items-center gap-3 group bg-black/20 hover:bg-black/40 px-4 py-1.5 rounded-lg transition-all border border-white/10"
              >
                <span className="text-[9px] font-black text-white/60 uppercase tracking-widest">Coupon:</span>
                <span className="text-[11px] font-mono font-black text-white tracking-widest">
                  {copied ? 'COPIED' : settings.code}
                </span>
              </button>
              
              <button 
                onClick={() => setVisible(false)}
                className="text-white/40 hover:text-white transition-colors"
              >
                <X className="w-4 h-4" />
              </button>
            </div>
          </div>
          {/* Subtle shimmer line */}
          <motion.div 
            animate={{ x: ['-100%', '100%'] }}
            transition={{ duration: 3, repeat: Infinity, ease: "linear" }}
            className="absolute bottom-0 left-0 w-1/3 h-[2px] bg-gradient-to-r from-transparent via-white/40 to-transparent"
          />
        </motion.div>
      )}
    </AnimatePresence>
  );
};

interface Plan {
  id: string;
  name: string;
  price: string;
  period: string;
  features: string[];
  popular: boolean;
  order: number;
  category: 'minecraft' | 'vps';
  subCategory?: 'premium' | 'budget';
}

const Navbar = ({ user, isAdmin, onAdminToggle, showAdmin }: { user: User | null, isAdmin: boolean, onAdminToggle: () => void, showAdmin: boolean }) => {
  const [scrolled, setScrolled] = useState(false);
  const [mobileMenuOpen, setMobileMenuOpen] = useState(false);

  useEffect(() => {
    const handleScroll = () => setScrolled(window.scrollY > 50);
    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  const navLinks = [
    { name: 'Home', href: '#' },
    { name: 'Infrastructure', href: '#features' },
    { name: 'Pricing', href: '#plans' },
    { name: 'Our Panel', href: PANEL_LINK, external: true },
    { name: 'Discord', href: DISCORD_LINK, external: true },
  ];

  return (
    <nav className={cn(
      "fixed top-0 left-0 right-0 z-[100] transition-all duration-700 px-4 sm:px-6 lg:px-8 py-6",
      scrolled ? "bg-black/80 backdrop-blur-3xl border-b border-white/5 py-4" : "bg-transparent"
    )}>
      <div className="max-w-7xl mx-auto flex items-center justify-between">
        <div className="flex items-center gap-12">
          <a href="/" className="flex items-center gap-3 group cursor-pointer">
            <div className="w-10 h-10 sm:w-12 sm:h-12 premium-gradient rounded-2xl flex items-center justify-center shadow-2xl shadow-brand-primary/40 group-hover:scale-110 transition-transform duration-500">
              <Server className="text-white w-5 h-5 sm:w-7 sm:h-7" />
            </div>
            <span className="text-2xl sm:text-3xl font-display font-black tracking-tighter text-white">Rexa<span className="text-brand-primary">Cloud</span></span>
          </a>
          
          <div className="hidden lg:flex items-center gap-10 text-[10px] font-black uppercase tracking-[0.4em] text-slate-400">
            {navLinks.map(link => (
              <a 
                key={link.name}
                href={link.href} 
                target={link.external ? "_blank" : undefined}
                rel={link.external ? "noopener noreferrer" : undefined}
                className="hover:text-white transition-colors"
              >
                {link.name}
              </a>
            ))}
          </div>
        </div>

        <div className="flex items-center gap-4 sm:gap-6">
          <div className="hidden sm:flex items-center gap-6">
            {user ? (
              <div className="flex items-center gap-6">
                {isAdmin && (
                  <button 
                    onClick={onAdminToggle}
                    className={cn(
                      "px-6 py-3 rounded-xl text-[10px] font-black uppercase tracking-[0.3em] transition-all",
                      showAdmin ? "bg-white text-black" : "bg-brand-primary/10 text-brand-primary border border-brand-primary/20 hover:bg-brand-primary/20"
                    )}
                  >
                    {showAdmin ? 'Exit Console' : 'Admin Console'}
                  </button>
                )}
                <div className="hidden lg:flex flex-col items-end">
                  <span className="text-[10px] font-black text-white uppercase tracking-widest">{user.displayName}</span>
                  <span className="text-[8px] font-bold text-brand-primary uppercase tracking-[0.2em]">{isAdmin ? 'Administrator' : 'Client'}</span>
                </div>
                <button 
                  onClick={() => auth.signOut()}
                  className="w-10 h-10 sm:w-12 sm:h-12 rounded-2xl bg-white/[0.03] border border-white/5 flex items-center justify-center text-slate-400 hover:text-white hover:bg-white/[0.08] transition-all luxury-border"
                >
                  <LogOut className="w-5 h-5" />
                </button>
              </div>
            ) : (
              <button 
                onClick={() => signInWithPopup(auth, new GoogleAuthProvider())}
                className="btn-premium bg-white/[0.03] border border-white/5 text-[10px] font-black uppercase tracking-[0.4em] px-8 py-4 rounded-2xl luxury-border"
              >
                Client Login
              </button>
            )}
          </div>

          {/* Mobile Menu Toggle */}
          <button 
            onClick={() => setMobileMenuOpen(!mobileMenuOpen)}
            className="lg:hidden w-10 h-10 sm:w-12 sm:h-12 rounded-2xl bg-white/[0.03] border border-white/5 flex items-center justify-center text-white hover:bg-white/[0.08] transition-all"
          >
            {mobileMenuOpen ? <X className="w-5 h-5" /> : <Menu className="w-5 h-5" />}
          </button>
        </div>
      </div>

      {/* Mobile Menu Overlay */}
      <AnimatePresence>
        {mobileMenuOpen && (
          <motion.div
            initial={{ opacity: 0, height: 0 }}
            animate={{ opacity: 1, height: 'auto' }}
            exit={{ opacity: 0, height: 0 }}
            className="lg:hidden absolute top-full left-0 right-0 bg-black/95 backdrop-blur-3xl border-b border-white/5 overflow-hidden"
          >
            <div className="px-8 py-12 space-y-8">
              {navLinks.map(link => (
                <a 
                  key={link.name}
                  href={link.href}
                  onClick={() => setMobileMenuOpen(false)}
                  target={link.external ? "_blank" : undefined}
                  rel={link.external ? "noopener noreferrer" : undefined}
                  className="block text-2xl font-display font-black text-white tracking-tighter hover:text-brand-primary transition-colors"
                >
                  {link.name}
                </a>
              ))}
              <div className="pt-8 border-t border-white/5 flex flex-col gap-6">
                {user ? (
                  <>
                    <div className="flex items-center gap-4">
                      <div className="w-12 h-12 rounded-full premium-gradient flex items-center justify-center text-white font-black">
                        {user.displayName?.[0]}
                      </div>
                      <div>
                        <p className="text-white font-black uppercase tracking-widest text-sm">{user.displayName}</p>
                        <p className="text-brand-primary font-bold uppercase tracking-[0.2em] text-[10px]">{isAdmin ? 'Administrator' : 'Client'}</p>
                      </div>
                    </div>
                    {isAdmin && (
                      <button 
                        onClick={() => {
                          onAdminToggle();
                          setMobileMenuOpen(false);
                        }}
                        className="w-full py-4 rounded-xl bg-brand-primary text-white text-[10px] font-black uppercase tracking-[0.4em]"
                      >
                        {showAdmin ? 'Exit Admin Console' : 'Open Admin Console'}
                      </button>
                    )}
                    <button 
                      onClick={() => auth.signOut()}
                      className="w-full py-4 rounded-xl bg-white/[0.03] border border-white/5 text-slate-400 text-[10px] font-black uppercase tracking-[0.4em]"
                    >
                      Sign Out
                    </button>
                  </>
                ) : (
                  <button 
                    onClick={() => signInWithPopup(auth, new GoogleAuthProvider())}
                    className="w-full py-5 rounded-2xl premium-gradient text-white text-[10px] font-black uppercase tracking-[0.4em] shadow-2xl shadow-brand-primary/40"
                  >
                    Client Login
                  </button>
                )}
              </div>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </nav>
  );
};

// ... Navbar component ...

const BackgroundDecorations = React.memo(() => {
  const streams = React.useMemo(() => [...Array(30)].map(() => ({
    left: `${Math.random() * 100}%`,
    delay: `${Math.random() * 10}s`,
    duration: `${3 + Math.random() * 5}s`,
    opacity: 0.05 + Math.random() * 0.15
  })), []);

  const shapes = React.useMemo(() => [...Array(12)].map(() => ({
    left: `${Math.random() * 100}%`,
    top: `${Math.random() * 100}%`,
    size: `${15 + Math.random() * 35}px`,
    delay: `${Math.random() * 5}s`,
    duration: `${10 + Math.random() * 20}s`,
    rotate: Math.random() * 360
  })), []);

  const codeLines = React.useMemo(() => [...Array(25)].map(() => ({
    left: `${Math.random() * 100}%`,
    top: `${Math.random() * 100}%`,
    delay: `${Math.random() * 20}s`,
    text: [
      '010101', 'EPYC', 'NVMe', '100G', 'DDoS', 'REXACLOUD', 'CORE', 'NODE', 'VPS',
      'INTEL', 'RYZEN', '99.9%', 'SLA', 'API', 'ROOT', 'SSH', 'PORT', 'PING', 'LATENCY'
    ][Math.floor(Math.random() * 19)]
  })), []);

  return (
    <div className="absolute inset-0 overflow-hidden pointer-events-none -z-10">
      {/* Deep Grid */}
      <div className="absolute inset-0 grid-mask opacity-[0.15]">
        <div className="w-full h-full texture-3d bg-[length:60px_60px]" />
      </div>

      {/* Scanning Line */}
      <motion.div 
        animate={{ top: ['-10%', '110%'] }}
        transition={{ duration: 15, repeat: Infinity, ease: "linear" }}
        className="absolute left-0 right-0 h-[2px] bg-gradient-to-r from-transparent via-brand-primary/20 to-transparent opacity-30 blur-sm"
      />

      {/* Circuit Lines */}
      <svg className="absolute inset-0 w-full h-full opacity-[0.05]" xmlns="http://www.w3.org/2000/svg">
        <pattern id="circuit" x="0" y="0" width="300" height="300" patternUnits="userSpaceOnUse">
          <path d="M 0 150 L 75 150 L 112 112 L 187 112 L 225 150 L 300 150" fill="none" stroke="var(--color-brand-primary)" strokeWidth="1" />
          <circle cx="75" cy="150" r="2" fill="var(--color-brand-primary)" />
          <circle cx="225" cy="150" r="2" fill="var(--color-brand-primary)" />
          <path d="M 150 0 L 150 75 L 112 112" fill="none" stroke="var(--color-brand-primary)" strokeWidth="1" />
          <circle cx="150" cy="75" r="2" fill="var(--color-brand-primary)" />
        </pattern>
        <rect width="100%" height="100%" fill="url(#circuit)" />
      </svg>

      {/* Floating Code Snippets */}
      {codeLines.map((line, i) => (
        <motion.div
          key={`code-${i}`}
          initial={{ opacity: 0 }}
          animate={{ opacity: [0, 0.2, 0] }}
          transition={{ duration: 10, repeat: Infinity, delay: i * 0.8 }}
          className="absolute font-mono text-[8px] font-black text-brand-primary tracking-[0.8em]"
          style={{ left: line.left, top: line.top }}
        >
          {line.text}
        </motion.div>
      ))}

      {/* Floating Geometric Shapes */}
      {shapes.map((shape, i) => (
        <motion.div
          key={`shape-${i}`}
          initial={{ opacity: 0, rotate: shape.rotate }}
          animate={{ 
            opacity: [0.03, 0.08, 0.03],
            rotate: shape.rotate + 360,
            y: [0, -30, 0],
            x: [0, 15, 0]
          }}
          transition={{ 
            duration: shape.duration, 
            repeat: Infinity, 
            ease: "easeInOut" 
          }}
          className="absolute border border-brand-primary/10 rounded-sm"
          style={{
            left: shape.left,
            top: shape.top,
            width: shape.size,
            height: shape.size,
          }}
        />
      ))}

      {/* Glow Orbs */}
      <div className="glow-orb top-[-25%] left-[-15%] animate-drift opacity-20" />
      <div className="glow-orb bottom-[-25%] right-[-15%] animate-drift opacity-20" style={{ animationDelay: '-5s' }} />
      <div className="glow-orb top-[35%] left-[35%] animate-drift opacity-[0.08]" style={{ animationDelay: '-12s', width: '900px', height: '900px' }} />
      
      <div className="scanline opacity-[0.03]" />
      
      {/* Data Streams */}
      {streams.map((stream, i) => (
        <div 
          key={`stream-${i}`}
          className="data-stream"
          style={{
            left: stream.left,
            animationDelay: stream.delay,
            animationDuration: stream.duration,
            opacity: stream.opacity
          }}
        />
      ))}

      {/* Vignette */}
      <div className="absolute inset-0 bg-[radial-gradient(circle_at_center,_transparent_0%,_black_100%)] opacity-50" />
    </div>
  );
});

const Hero = () => {
  const [mousePos, setMousePos] = useState({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setMousePos({
        x: (e.clientX / window.innerWidth - 0.5) * 20,
        y: (e.clientY / window.innerHeight - 0.5) * 20
      });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  return (
    <section className="relative pt-32 pb-20 lg:pt-64 lg:pb-48 overflow-hidden bg-black">
      <BackgroundDecorations />
      
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 text-center relative">
        <motion.div
          initial={{ opacity: 0, y: 40 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ duration: 1, ease: [0.16, 1, 0.3, 1] }}
        >
          <div className="inline-flex items-center gap-3 px-6 py-3 mb-12 rounded-2xl bg-white/[0.03] border border-white/5 text-[10px] font-black tracking-[0.4em] uppercase text-brand-primary shadow-2xl luxury-border">
            <span className="relative flex h-2 w-2">
              <span className="animate-ping absolute inline-flex h-full w-full rounded-full bg-brand-primary opacity-75"></span>
              <span className="relative inline-flex rounded-full h-2 w-2 bg-brand-primary"></span>
            </span>
            Next-Gen Infrastructure • Tier-4 Certified
          </div>
          
          <h1 className="text-7xl lg:text-[11rem] font-display font-black mb-12 leading-[0.8] tracking-tighter text-white">
            Raw <br />
            <span className="shimmer-text">Power.</span>
          </h1>
          
          <p className="text-lg lg:text-2xl text-slate-400 mb-16 max-w-3xl mx-auto leading-relaxed font-medium tracking-tight">
            Deploy mission-critical applications on <span className="text-white">AMD EPYC™</span> hardware. 
            No virtualization overhead. No compromises. Just pure performance.
          </p>
          
          <div className="flex flex-col sm:flex-row items-center justify-center gap-8">
            <motion.a 
              whileHover={{ scale: 1.05, y: -4 }}
              whileTap={{ scale: 0.95 }}
              href="#plans" 
              className="btn-premium premium-gradient min-w-[280px] text-lg shadow-2xl shadow-brand-primary/30 py-5 rounded-2xl"
            >
              Deploy Instance <ChevronRight className="w-5 h-5" />
            </motion.a>
            <motion.a 
              whileHover={{ backgroundColor: 'rgba(255,255,255,0.08)', y: -4 }}
              href={DISCORD_LINK} 
              target="_blank" 
              rel="noopener noreferrer" 
              className="btn-premium bg-white/[0.03] border border-white/5 min-w-[280px] text-lg py-5 rounded-2xl luxury-border"
            >
              <MessageSquare className="w-5 h-5" /> Technical Support
            </motion.a>
          </div>
        </motion.div>

        {/* Stats Section */}
        <div className="grid grid-cols-2 md:grid-cols-4 gap-8 mt-32 max-w-5xl mx-auto">
          {[
            { label: 'Uptime', value: '99.99%', sub: 'Guaranteed' },
            { label: 'Latency', value: '<1ms', sub: 'Internal' },
            { label: 'Network', value: '100Gbps', sub: 'Backbone' },
            { label: 'Support', value: '24/7', sub: 'Expert' }
          ].map((stat, i) => (
            <motion.div
              key={i}
              initial={{ opacity: 0, y: 20 }}
              whileInView={{ opacity: 1, y: 0 }}
              transition={{ delay: i * 0.1 }}
              className="glass-card p-8 luxury-border group"
            >
              <p className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 mb-2 group-hover:text-brand-primary transition-colors">{stat.label}</p>
              <p className="text-3xl font-display font-black text-white mb-1">{stat.value}</p>
              <p className="text-[8px] font-bold uppercase tracking-widest text-slate-600">{stat.sub}</p>
            </motion.div>
          ))}
        </div>

        <motion.div
          initial={{ opacity: 0, scale: 0.98, y: 60 }}
          animate={{ 
            opacity: 1, 
            scale: 1, 
            y: 0,
            rotateX: -mousePos.y * 0.5,
            rotateY: mousePos.x * 0.5
          }}
          transition={{ duration: 1.2, delay: 0.4, ease: [0.16, 1, 0.3, 1] }}
          className="mt-32 relative max-w-6xl mx-auto group card-3d"
        >
          <div className="absolute -inset-4 bg-brand-primary/10 rounded-[3rem] blur-3xl opacity-0 group-hover:opacity-100 transition-opacity duration-1000" />
          <div className="glass-card p-3 relative overflow-hidden inner-3d">
            <div className="aspect-[21/9] rounded-[1.8rem] overflow-hidden bg-black flex items-center justify-center relative">
              <div className="absolute inset-0 bg-[url('https://www.transparenttextures.com/patterns/carbon-fibre.png')] opacity-10" />
              <div className="absolute inset-0 bg-gradient-to-t from-black via-transparent to-transparent" />
              
              <div className="relative z-10 flex flex-col items-center gap-8">
                <div className="w-24 h-24 premium-gradient rounded-[2rem] flex items-center justify-center animate-float shadow-2xl shadow-brand-primary/50">
                  <Cpu className="w-12 h-12 text-white" />
                </div>
                <div className="text-center">
                  <div className="flex items-center justify-center gap-3 mb-4">
                    <div className="w-2.5 h-2.5 rounded-full bg-blue-500 animate-glow" />
                    <span className="text-xs font-mono font-black text-blue-400 tracking-[0.4em] uppercase">AMD EPYC™ 7R13 • ACTIVE</span>
                  </div>
                  <p className="text-3xl lg:text-4xl font-display font-black tracking-tighter text-white">Engineered for Dominance.</p>
                </div>
              </div>

              {/* Decorative elements */}
              <div className="absolute top-10 right-10 font-mono text-[10px] text-slate-700 text-right leading-relaxed tracking-widest uppercase">
                LOAD: 0.04%<br />
                MEM: 1.2 / 512GB<br />
                NET: 100GBPS
              </div>
              <div className="absolute bottom-10 left-10 font-mono text-[10px] text-slate-700 leading-relaxed tracking-widest uppercase">
                STATUS: OPTIMAL<br />
                TEMP: 32°C<br />
                UPTIME: 99.99%
              </div>
            </div>
          </div>
        </motion.div>
      </div>
    </section>
  );
};

const Features = () => {
  const features = [
    {
      icon: <Cpu className="w-7 h-7 text-brand-primary" />,
      title: "AMD EPYC™ 7R13",
      description: "Enterprise-grade processors with high clock speeds for demanding workloads."
    },
    {
      icon: <Zap className="w-7 h-7 text-blue-400" />,
      title: "NVMe Gen4 Storage",
      description: "Experience ultra-low latency with our high-performance NVMe storage arrays."
    },
    {
      icon: <Shield className="w-7 h-7 text-blue-600" />,
      title: "DDoS Mitigation",
      description: "Advanced multi-layered protection filtering malicious traffic at the edge."
    },
    {
      icon: <Settings className="w-7 h-7 text-slate-500" />,
      title: "Intuitive Control",
      description: "Manage your infrastructure with our streamlined, high-performance control panel."
    },
    {
      icon: <Globe className="w-7 h-7 text-blue-300" />,
      title: "Global Connectivity",
      description: "Low-latency networking with premium transit providers across the globe."
    },
    {
      icon: <Check className="w-7 h-7 text-blue-500" />,
      title: "Dedicated Resources",
      description: "We guarantee 100% resource allocation. No overselling, no compromises."
    }
  ];

  return (
    <section id="features" className="py-48 bg-black relative overflow-hidden">
      <BackgroundDecorations />
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex flex-col lg:flex-row items-end justify-between mb-32 gap-12">
          <div className="max-w-2xl">
            <h2 className="text-xs font-black tracking-[0.5em] uppercase text-brand-primary mb-6">Core Infrastructure</h2>
            <h3 className="text-5xl lg:text-8xl font-display font-black leading-[0.9] tracking-tighter text-white">
              Built for <br />
              <span className="shimmer-text">Reliability.</span>
            </h3>
          </div>
          <p className="text-slate-500 max-w-md text-xl leading-relaxed font-medium tracking-tight">
            We've engineered our cloud platform from the ground up to provide a 
            stable, high-performance environment for your most critical projects.
          </p>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
          {features.map((feature, index) => (
            <motion.div
              key={index}
              initial={{ opacity: 0, y: 20 }}
              whileInView={{ opacity: 1, y: 0 }}
              transition={{ delay: index * 0.1 }}
              viewport={{ once: true }}
              className="glass-card p-12 group relative overflow-hidden luxury-border"
            >
              <div className="absolute -right-12 -top-12 w-32 h-32 bg-brand-primary/5 rounded-full blur-3xl group-hover:bg-brand-primary/10 transition-colors" />
              <div className="mb-10 p-5 w-fit rounded-2xl bg-white/[0.03] border border-white/5 group-hover:scale-110 group-hover:rotate-3 transition-transform duration-500 shadow-inner">
                {feature.icon}
              </div>
              <h4 className="text-2xl font-display font-black mb-4 tracking-tighter text-white">{feature.title}</h4>
              <p className="text-slate-500 leading-relaxed font-medium tracking-tight group-hover:text-slate-400 transition-colors">{feature.description}</p>
            </motion.div>
          ))}
        </div>
      </div>
    </section>
  );
};


const Gallery = () => {
  const images = [
    { src: 'https://images.unsplash.com/photo-1558494949-ef010cbdcc31?auto=format&fit=crop&q=80&w=2000', title: 'Network Infrastructure' },
    { src: 'https://images.unsplash.com/photo-1563986768609-322da13575f3?auto=format&fit=crop&q=80&w=2000', title: 'Server Rack 01' },
    { src: 'https://images.unsplash.com/photo-1518770660439-4636190af475?auto=format&fit=crop&q=80&w=2000', title: 'Control Panel' },
    { src: 'https://images.unsplash.com/photo-1550751827-4bd374c3f58b?auto=format&fit=crop&q=80&w=2000', title: 'Data Center Cooling' },
    { src: 'https://images.unsplash.com/photo-1451187580459-43490279c0fa?auto=format&fit=crop&q=80&w=2000', title: 'Hardware Monitoring' },
    { src: 'https://images.unsplash.com/photo-1562408590-e32931084e23?auto=format&fit=crop&q=80&w=2000', title: 'Security Systems' }
  ];

  return (
    <section id="gallery" className="py-48 bg-black relative overflow-hidden">
      <BackgroundDecorations />
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="text-center mb-32">
          <h2 className="text-xs font-black tracking-[0.5em] uppercase text-brand-primary mb-6">Infrastructure Gallery</h2>
          <h3 className="text-5xl lg:text-8xl font-display font-black tracking-tighter text-white">
            Visualizing <span className="shimmer-text">Excellence.</span>
          </h3>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-10">
          {images.map((img, index) => (
            <motion.div
              key={index}
              initial={{ opacity: 0, scale: 0.98 }}
              whileInView={{ opacity: 1, scale: 1 }}
              transition={{ delay: index * 0.1, duration: 0.8 }}
              viewport={{ once: true }}
              className="group relative aspect-[4/3] rounded-[3rem] overflow-hidden border border-white/5 bg-black shadow-2xl luxury-border"
            >
              <img 
                src={img.src} 
                alt={img.title} 
                className="w-full h-full object-cover transition-transform duration-1000 group-hover:scale-110 opacity-40 group-hover:opacity-100"
                referrerPolicy="no-referrer"
              />
              <div className="absolute inset-0 bg-gradient-to-t from-black via-black/10 to-transparent opacity-90 group-hover:opacity-40 transition-opacity duration-700 flex items-end p-12">
                <div className="translate-y-4 group-hover:translate-y-0 transition-transform duration-700">
                  <p className="text-brand-primary text-[10px] font-black tracking-[0.5em] uppercase mb-3">Infrastructure</p>
                  <h4 className="text-3xl font-display font-black text-white tracking-tighter">{img.title}</h4>
                </div>
              </div>
            </motion.div>
          ))}
        </div>
      </div>
    </section>
  );
};

const TrustedBy = () => {
  return (
    <section className="py-24 relative overflow-hidden bg-black/50 border-y border-white/5">
      <div className="max-w-7xl mx-auto px-4">
        <div className="flex flex-col items-center gap-12">
          <div className="text-center space-y-4">
            <motion.div
              initial={{ opacity: 0, y: 20 }}
              whileInView={{ opacity: 1, y: 0 }}
              viewport={{ once: true }}
              className="flex items-center justify-center gap-3 mb-6"
            >
              <div className="h-[1px] w-12 bg-gradient-to-r from-transparent to-brand-primary/50" />
              <span className="text-brand-primary font-black uppercase tracking-[0.5em] text-[10px]">Ecosystem</span>
              <div className="h-[1px] w-12 bg-gradient-to-l from-transparent to-brand-primary/50" />
            </motion.div>
            <h2 className="text-3xl md:text-5xl font-black uppercase tracking-tighter text-white">
              Trusted by <span className="text-brand-primary">thousands</span> of users
            </h2>
            <p className="text-slate-400 font-bold uppercase tracking-widest text-[10px] max-w-md mx-auto">
              Powering the next generation of digital experiences across the globe.
            </p>
          </div>
          
          <div className="grid grid-cols-2 md:grid-cols-5 gap-8 w-full opacity-30 grayscale hover:grayscale-0 transition-all duration-700">
            {['INTEL', 'AMD', 'SAMSUNG', 'NVIDIA', 'CISCO'].map((brand) => (
              <div key={brand} className="flex items-center justify-center p-8 glass-card luxury-border group hover:bg-white/5 transition-all">
                <span className="text-xl lg:text-2xl font-display font-black tracking-tighter text-white group-hover:text-brand-primary transition-colors cursor-default">{brand}</span>
              </div>
            ))}
          </div>
        </div>
      </div>
    </section>
  );
};

const WhyChooseUs = () => {
  const reasons = [
    { title: "Enterprise Hardware", desc: "We only use the latest AMD EPYC processors and NVMe Gen4 SSDs.", icon: <Cpu className="w-6 h-6" /> },
    { title: "99.9% Uptime", desc: "Our Tier-4 data centers ensure your services stay online 24/7/365.", icon: <Shield className="w-6 h-6" /> },
    { title: "Instant Setup", desc: "Your server is provisioned within seconds of payment confirmation.", icon: <Zap className="w-6 h-6" /> },
    { title: "Expert Support", desc: "Our technical team is available around the clock to assist you.", icon: <Headphones className="w-6 h-6" /> }
  ];

  return (
    <section className="py-48 bg-black relative overflow-hidden">
      <BackgroundDecorations />
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="grid grid-cols-1 lg:grid-cols-2 gap-24 items-center">
          <div>
            <h2 className="text-xs font-black tracking-[0.5em] uppercase text-brand-primary mb-6">Why RexaCloud?</h2>
            <h3 className="text-5xl lg:text-7xl font-display font-black tracking-tighter text-white mb-10">
              The Gold Standard in <span className="shimmer-text">Cloud Hosting.</span>
            </h3>
            <p className="text-slate-500 text-xl leading-relaxed mb-12 font-medium">
              We don't just provide servers; we provide the foundation for your digital success. 
              Our infrastructure is optimized for speed, security, and scalability.
            </p>
            <div className="space-y-8">
              {reasons.map((reason, i) => (
                <div key={i} className="flex gap-6 group">
                  <div className="w-14 h-14 rounded-2xl bg-white/[0.03] border border-white/5 flex items-center justify-center text-brand-primary group-hover:scale-110 transition-transform luxury-border">
                    {reason.icon}
                  </div>
                  <div>
                    <h4 className="text-xl font-display font-black text-white mb-2">{reason.title}</h4>
                    <p className="text-slate-500 font-medium leading-relaxed">{reason.desc}</p>
                  </div>
                </div>
              ))}
            </div>
          </div>
          <div className="relative">
            <div className="absolute -inset-10 bg-brand-primary/10 blur-[120px] rounded-full" />
            <div className="glass-card p-4 luxury-border relative overflow-hidden group">
              <img 
                src="https://images.unsplash.com/photo-1558494949-ef010cbdcc31?auto=format&fit=crop&q=80&w=2000" 
                alt="Infrastructure" 
                className="rounded-[2rem] opacity-50 group-hover:opacity-80 transition-opacity duration-700"
                referrerPolicy="no-referrer"
              />
              <div className="absolute inset-0 flex items-center justify-center">
                <div className="w-24 h-24 premium-gradient rounded-full flex items-center justify-center animate-pulse shadow-2xl shadow-brand-primary/50">
                  <Server className="w-10 h-10 text-white" />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </section>
  );
};

const Plans = ({ plans }: { plans: Plan[] }) => {
  const [activeCategory, setActiveCategory] = useState<'minecraft' | 'vps'>('minecraft');
  const [activeSubCategory, setActiveSubCategory] = useState<'premium' | 'budget'>('premium');
  
  const filteredPlans = plans
    .filter(p => {
      const catMatch = (p.category || 'minecraft') === activeCategory;
      const subCatMatch = (p.subCategory || 'premium') === activeSubCategory;
      return catMatch && subCatMatch;
    })
    .sort((a, b) => a.order - b.order);

  return (
    <section id="plans" className="py-48 bg-black relative overflow-hidden">
      <BackgroundDecorations />
      <div className="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 w-full h-full -z-10">
        <div className="absolute top-0 left-0 w-full h-full bg-gradient-to-b from-transparent via-brand-primary/[0.05] to-transparent" />
      </div>

      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="text-center mb-32">
          <motion.div
            initial={{ opacity: 0, scale: 0.98 }}
            whileInView={{ opacity: 1, scale: 1 }}
            viewport={{ once: true }}
          >
            <h2 className="text-xs font-black tracking-[0.5em] uppercase text-brand-primary mb-8">Pricing & Infrastructure</h2>
            <h3 className="text-5xl lg:text-8xl font-display font-black tracking-tighter mb-10 text-white">
              Choose Your <span className="shimmer-text">Power.</span>
            </h3>
            
            {/* Main Category Toggle */}
            <div className="flex items-center justify-center gap-6 mt-16">
              <button 
                onClick={() => setActiveCategory('minecraft')}
                className={cn(
                  "px-10 py-4 rounded-2xl font-black text-[10px] uppercase tracking-[0.4em] transition-all duration-700 border luxury-border",
                  activeCategory === 'minecraft' 
                    ? "premium-gradient border-transparent shadow-2xl shadow-brand-primary/40 text-white scale-105" 
                    : "bg-white/[0.02] border-white/5 text-slate-500 hover:text-white"
                )}
              >
                Minecraft Hosting
              </button>
              <button 
                onClick={() => setActiveCategory('vps')}
                className={cn(
                  "px-10 py-4 rounded-2xl font-black text-[10px] uppercase tracking-[0.4em] transition-all duration-700 border luxury-border",
                  activeCategory === 'vps' 
                    ? "premium-gradient border-transparent shadow-2xl shadow-brand-primary/40 text-white scale-105" 
                    : "bg-white/[0.02] border-white/5 text-slate-500 hover:text-white"
                )}
              >
                VPS Hosting
              </button>
            </div>

            {/* Sub-category Toggle */}
            <motion.div 
              initial={{ opacity: 0, y: 10 }}
              animate={{ opacity: 1, y: 0 }}
              className="flex items-center justify-center gap-4 mt-12"
            >
              <button 
                onClick={() => setActiveSubCategory('premium')}
                className={cn(
                  "px-8 py-3 rounded-xl font-black text-[8px] uppercase tracking-[0.3em] transition-all duration-500 border",
                  activeSubCategory === 'premium'
                    ? "bg-brand-primary/20 border-brand-primary/50 text-brand-primary shadow-lg shadow-brand-primary/10"
                    : "bg-white/[0.01] border-white/5 text-slate-600 hover:text-slate-400"
                )}
              >
                {activeCategory === 'minecraft' ? 'Premium Nodes' : 'Enterprise VPS'}
              </button>
              <button 
                onClick={() => setActiveSubCategory('budget')}
                className={cn(
                  "px-8 py-3 rounded-xl font-black text-[8px] uppercase tracking-[0.3em] transition-all duration-500 border",
                  activeSubCategory === 'budget'
                    ? "bg-brand-primary/20 border-brand-primary/50 text-brand-primary shadow-lg shadow-brand-primary/10"
                    : "bg-white/[0.01] border-white/5 text-slate-600 hover:text-slate-400"
                )}
              >
                {activeCategory === 'minecraft' ? 'Budget Nodes' : 'Standard VPS'}
              </button>
            </motion.div>
          </motion.div>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-12">
          <AnimatePresence mode="wait">
            {filteredPlans.map((plan, index) => (
              <motion.div
                key={plan.id}
                initial={{ opacity: 0, y: 40, scale: 0.95 }}
                animate={{ opacity: 1, y: 0, scale: 1 }}
                exit={{ opacity: 0, y: -20, scale: 0.95 }}
                transition={{ delay: index * 0.1, duration: 0.8, ease: [0.16, 1, 0.3, 1] }}
                whileHover={{ y: -16 }}
                className={cn(
                  "glass-card p-1 relative group flex flex-col min-h-[650px] luxury-border",
                  plan.popular && "border-brand-primary/30 shadow-2xl shadow-brand-primary/20"
                )}
              >
                {plan.popular && (
                  <div className="absolute -top-6 left-1/2 -translate-x-1/2 premium-gradient px-8 py-2 rounded-full text-[10px] font-black uppercase tracking-[0.4em] z-10 shadow-2xl">
                    Most Popular
                  </div>
                )}
                
                <div className="p-12 flex-grow flex flex-col">
                  <div className="mb-12">
                    <div className="flex items-center gap-3 mb-6">
                      <div className="w-2.5 h-2.5 rounded-full bg-brand-primary animate-glow" />
                      <span className="text-[10px] font-black uppercase tracking-[0.5em] text-slate-500">Tier 0{index + 1}</span>
                    </div>
                    <h4 className="text-4xl font-display font-black mb-4 tracking-tighter group-hover:text-brand-primary transition-colors text-white">{plan.name}</h4>
                    <div className="flex items-baseline gap-2">
                      <span className="text-7xl font-display font-black tracking-tighter text-white">₹{plan.price}</span>
                      <span className="text-slate-600 font-bold tracking-widest uppercase text-[10px]">{plan.period}</span>
                    </div>
                  </div>

                  <div className="space-y-6 mb-16 flex-grow">
                    {plan.features.map((feature, idx) => (
                      <div key={idx} className="flex items-center gap-5 text-slate-400 font-medium tracking-tight group/item">
                        <div className="w-7 h-7 rounded-full bg-brand-primary/5 flex items-center justify-center shrink-0 border border-white/5 group-hover/item:border-brand-primary/30 transition-colors">
                          <Check className="w-4 h-4 text-brand-primary" />
                        </div>
                        <span className="text-sm group-hover/item:text-slate-300 transition-colors">{feature}</span>
                      </div>
                    ))}
                  </div>

                  <motion.a
                    whileHover={{ scale: 1.05 }}
                    whileTap={{ scale: 0.95 }}
                    href={DISCORD_LINK}
                    target="_blank"
                    rel="noopener noreferrer"
                    className={cn(
                      "btn-premium w-full text-[10px] font-black uppercase tracking-[0.4em] shadow-2xl py-5 rounded-2xl",
                      plan.popular 
                        ? "premium-gradient text-white shadow-brand-primary/40" 
                        : "bg-white/[0.03] border border-white/5 hover:bg-white/[0.06] text-white"
                    )}
                  >
                    Deploy Instance <ArrowRight className="w-5 h-5 group-hover:translate-x-2 transition-transform" />
                  </motion.a>
                </div>
              </motion.div>
            ))}
          </AnimatePresence>
        </div>
        
        {filteredPlans.length === 0 && (
          <div className="text-center py-32">
            <div className="w-24 h-24 bg-white/[0.02] border border-white/5 rounded-[2rem] flex items-center justify-center mx-auto mb-10 luxury-border">
              <Server className="w-12 h-12 text-slate-700" />
            </div>
            <p className="text-slate-500 font-black tracking-[0.5em] uppercase text-[12px] mb-4">No infrastructure available.</p>
            <p className="text-slate-700 font-bold tracking-widest uppercase text-[10px]">
              {activeCategory === 'minecraft' && activeSubCategory === 'budget' 
                ? "Budget nodes are currently being provisioned. Stay tuned." 
                : "This sector is currently offline."}
            </p>
          </div>
        )}
      </div>
    </section>
  );
};

const AdminDashboard = ({ plans, isAdmin, bannerSettings }: { plans: Plan[], isAdmin: boolean, bannerSettings: BannerSettings | null }) => {
  const [isAdding, setIsAdding] = useState(false);
  const [editingPlan, setEditingPlan] = useState<Plan | null>(null);
  const [activeTab, setActiveTab] = useState<'plans' | 'settings'>('plans');
  const [adminFilter, setAdminFilter] = useState<'all' | 'minecraft' | 'vps' | 'minecraft-premium' | 'minecraft-budget'>('all');
  const [formData, setFormData] = useState({
    name: '',
    price: '',
    period: '/mo',
    features: '',
    popular: false,
    order: 0,
    category: 'minecraft' as 'minecraft' | 'vps',
    subCategory: 'premium' as 'premium' | 'budget'
  });

  const [bannerForm, setBannerForm] = useState<BannerSettings>(bannerSettings || {
    text: 'Launch Offer: 30% OFF all plans',
    code: 'REXA30',
    discount: '30%',
    active: true
  });

  useEffect(() => {
    if (bannerSettings) {
      setBannerForm(bannerSettings);
    }
  }, [bannerSettings]);

  if (!isAdmin) return null;

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    const planData = {
      ...formData,
      features: formData.features.split(',').map(f => f.trim()).filter(f => f !== ''),
      order: Number(formData.order)
    };

    try {
      if (editingPlan) {
        await updateDoc(doc(db, 'plans', editingPlan.id), planData);
        setEditingPlan(null);
      } else {
        await addDoc(collection(db, 'plans'), planData);
        setIsAdding(false);
      }
      setFormData({ name: '', price: '', period: '/mo', features: '', popular: false, order: 0, category: 'minecraft', subCategory: 'premium' });
    } catch (error) {
      console.error("Error saving plan:", error);
    }
  };

  const handleBannerSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    try {
      await setDoc(doc(db, 'settings', 'banner'), bannerForm);
      alert("Banner settings updated successfully!");
    } catch (error) {
      console.error("Error saving banner settings:", error);
    }
  };

  const seedMinecraftPlans = async () => {
    const mcPlans = [
      { name: "Dirt Plan [STARTER]", price: "99", period: "/mo", features: ["2GB RAM", "CPU: 1 vCore", "25GB NVMe SSD", "1Gbps Network", "Best for small SMP"], popular: false, order: 1, category: 'minecraft', subCategory: 'premium' },
      { name: "Dirt Plan+ [POPULAR]", price: "199", period: "/mo", features: ["4GB RAM", "CPU: 2 vCore", "40GB NVMe SSD", "1Gbps Network", "Smooth small server performance"], popular: true, order: 2, category: 'minecraft', subCategory: 'premium' },
      { name: "Coal Plan [PLUS]", price: "299", period: "/mo", features: ["6GB RAM", "CPU: 2 vCore (High Priority)", "60GB NVMe SSD", "1Gbps Network", "Better stability & plugins"], popular: false, order: 3, category: 'minecraft', subCategory: 'premium' },
      { name: "Coal Plan+ [ADVANCED]", price: "399", period: "/mo", features: ["8GB RAM", "CPU: 3 vCore", "80GB NVMe SSD", "2Gbps Network", "Mid-size community servers"], popular: false, order: 4, category: 'minecraft', subCategory: 'premium' },
      { name: "Redstone Plan [PRO]", price: "499", period: "/mo", features: ["10GB RAM", "CPU: 4 vCore", "100GB NVMe SSD", "2Gbps Network", "Lag-free gameplay"], popular: false, order: 5, category: 'minecraft', subCategory: 'premium' },
      { name: "Redstone Plan+ [POWER]", price: "599", period: "/mo", features: ["12GB RAM", "CPU: 5 vCore", "120GB NVMe SSD", "2Gbps Network", "Heavy plugin usage"], popular: false, order: 6, category: 'minecraft', subCategory: 'premium' },
      { name: "Gold Plan [MOST POPULAR 🌟]", price: "799", period: "/mo", features: ["16GB RAM", "CPU: 6 vCore (Dedicated Threads)", "160GB NVMe Gen4 SSD", "5Gbps Network", "Modpacks & heavy load"], popular: true, order: 7, category: 'minecraft', subCategory: 'premium' },
      { name: "Diamond Plan [ULTRA]", price: "999", period: "/mo", features: ["20GB RAM", "CPU: 7 vCore Dedicated", "200GB NVMe Gen4 SSD", "5Gbps Network", "High player base"], popular: false, order: 8, category: 'minecraft', subCategory: 'premium' },
      { name: "Netherite Plan [EXTREME]", price: "1199", period: "/mo", features: ["24GB RAM", "CPU: 8 vCore Dedicated", "240GB NVMe Gen4 SSD", "5Gbps Network", "Large communities"], popular: false, order: 9, category: 'minecraft', subCategory: 'premium' },
      { name: "Obsidian Plan [ENTERPRISE 💎]", price: "1399", period: "/mo", features: ["28GB RAM", "CPU: 10 vCore Dedicated", "300GB NVMe Gen4 SSD", "10Gbps Network", "Networks & proxy setups"], popular: false, order: 10, category: 'minecraft', subCategory: 'premium' }
    ];

    if (window.confirm("Seed Minecraft Premium plans? This will add 10 new plans.")) {
      try {
        for (const plan of mcPlans) {
          await addDoc(collection(db, 'plans'), plan);
        }
        alert("Minecraft Premium plans seeded successfully!");
      } catch (error) {
        console.error("Error seeding plans:", error);
      }
    }
  };

  const seedMinecraftBudgetPlans = async () => {
    const mcPlans = [
      { name: "Dirt Budget", price: "49", period: "/mo", features: ["1GB RAM", "CPU: 1 vCore", "10GB NVMe SSD", "1Gbps Network", "Entry level hosting"], popular: false, order: 1, category: 'minecraft', subCategory: 'budget' },
      { name: "Coal Budget", price: "89", period: "/mo", features: ["2GB RAM", "CPU: 1 vCore", "20GB NVMe SSD", "1Gbps Network", "Small SMP"], popular: true, order: 2, category: 'minecraft', subCategory: 'budget' },
      { name: "Iron Budget", price: "149", period: "/mo", features: ["4GB RAM", "CPU: 2 vCore", "40GB NVMe SSD", "1Gbps Network", "Standard performance"], popular: false, order: 3, category: 'minecraft', subCategory: 'budget' }
    ];

    if (window.confirm("Seed Minecraft Budget plans? This will add 3 new plans.")) {
      try {
        for (const plan of mcPlans) {
          await addDoc(collection(db, 'plans'), plan);
        }
        alert("Minecraft Budget plans seeded successfully!");
      } catch (error) {
        console.error("Error seeding plans:", error);
      }
    }
  };

  const seedVPSPlans = async () => {
    const vpsPlans = [
      { name: "VPS Starter", price: "299", period: "/mo", features: ["2GB RAM", "CPU: 1 vCore EPYC", "40GB NVMe SSD", "10Gbps Network", "Linux/Windows support"], popular: false, order: 1, category: 'vps', subCategory: 'premium' },
      { name: "VPS Pro", price: "549", period: "/mo", features: ["4GB RAM", "CPU: 2 vCore EPYC", "80GB NVMe SSD", "10Gbps Network", "High performance"], popular: true, order: 2, category: 'vps', subCategory: 'premium' },
      { name: "VPS Ultra", price: "999", period: "/mo", features: ["8GB RAM", "CPU: 4 vCore EPYC", "160GB NVMe SSD", "10Gbps Network", "Enterprise grade"], popular: false, order: 3, category: 'vps', subCategory: 'premium' }
    ];

    if (window.confirm("Seed VPS plans? This will add 3 new plans.")) {
      try {
        for (const plan of vpsPlans) {
          await addDoc(collection(db, 'plans'), plan);
        }
        alert("VPS plans seeded successfully!");
      } catch (error) {
        console.error("Error seeding plans:", error);
      }
    }
  };

  const handleDelete = async (id: string) => {
    if (window.confirm("Are you sure you want to delete this infrastructure instance?")) {
      try {
        await deleteDoc(doc(db, 'plans', id));
      } catch (error) {
        console.error("Error deleting plan:", error);
      }
    }
  };

  const filteredAdminPlans = plans
    .filter(p => {
      if (adminFilter === 'all') return true;
      if (adminFilter === 'minecraft') return (p.category || 'minecraft') === 'minecraft';
      if (adminFilter === 'vps') return (p.category || 'minecraft') === 'vps';
      if (adminFilter === 'minecraft-premium') return (p.category || 'minecraft') === 'minecraft' && (p.subCategory || 'premium') === 'premium';
      if (adminFilter === 'minecraft-budget') return (p.category || 'minecraft') === 'minecraft' && (p.subCategory || 'premium') === 'budget';
      return true;
    })
    .sort((a, b) => a.order - b.order);

  return (
    <section id="admin" className="min-h-screen pt-48 pb-24 bg-black relative overflow-hidden">
      <BackgroundDecorations />
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="flex flex-col md:flex-row items-start md:items-center justify-between mb-24 gap-12">
          <div>
            <h2 className="text-xs font-black tracking-[0.5em] uppercase text-brand-primary mb-6">Management Interface</h2>
            <h3 className="text-5xl lg:text-8xl font-display font-black tracking-tighter text-white">
              Admin <span className="shimmer-text">Console.</span>
            </h3>
          </div>
          <div className="flex gap-4">
            <button 
              onClick={() => setActiveTab('plans')}
              className={cn(
                "px-8 py-4 rounded-xl text-[10px] font-black uppercase tracking-[0.3em] transition-all",
                activeTab === 'plans' ? "bg-white text-black" : "bg-white/[0.03] text-slate-400 border border-white/5"
              )}
            >
              Infrastructure
            </button>
            <button 
              onClick={() => setActiveTab('settings')}
              className={cn(
                "px-8 py-4 rounded-xl text-[10px] font-black uppercase tracking-[0.3em] transition-all",
                activeTab === 'settings' ? "bg-white text-black" : "bg-white/[0.03] text-slate-400 border border-white/5"
              )}
            >
              Banner Settings
            </button>
          </div>
        </div>

        {/* Admin Stats */}
        <div className="grid grid-cols-2 md:grid-cols-4 gap-8 mb-24">
          {[
            { label: 'Total Nodes', value: plans.length },
            { label: 'Minecraft', value: plans.filter(p => p.category === 'minecraft').length },
            { label: 'VPS', value: plans.filter(p => p.category === 'vps').length },
            { label: 'Popular', value: plans.filter(p => p.popular).length }
          ].map((stat, i) => (
            <div key={i} className="glass-card p-8 luxury-border">
              <p className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 mb-2">{stat.label}</p>
              <p className="text-4xl font-display font-black text-white">{stat.value}</p>
            </div>
          ))}
        </div>

        {activeTab === 'plans' ? (
          <>
            <div className="flex flex-col md:flex-row justify-between items-center mb-12 gap-6">
              <div className="flex flex-wrap gap-4">
                {(['all', 'minecraft', 'vps', 'minecraft-premium', 'minecraft-budget'] as const).map(filter => (
                  <button
                    key={filter}
                    onClick={() => setAdminFilter(filter)}
                    className={cn(
                      "px-6 py-2 rounded-lg text-[8px] font-black uppercase tracking-[0.2em] transition-all",
                      adminFilter === filter ? "bg-brand-primary text-white" : "bg-white/[0.03] text-slate-500 border border-white/5"
                    )}
                  >
                    {filter.replace('-', ' ')}
                  </button>
                ))}
              </div>
              <button 
                onClick={() => {
                  setIsAdding(!isAdding);
                  setEditingPlan(null);
                  setFormData({ name: '', price: '', period: '/mo', features: '', popular: false, order: 0, category: 'minecraft', subCategory: 'premium' });
                }}
                className="btn-premium premium-gradient px-12 py-5 rounded-2xl font-black text-[10px] uppercase tracking-[0.4em] shadow-2xl shadow-brand-primary/40"
              >
                {isAdding || editingPlan ? <X className="w-5 h-5" /> : <Plus className="w-5 h-5" />}
                {isAdding || editingPlan ? 'Close Editor' : 'Provision New Node'}
              </button>
            </div>

            {(isAdding || editingPlan) && (
              <motion.div 
                initial={{ opacity: 0, y: 20 }}
                animate={{ opacity: 1, y: 0 }}
                className="glass-card p-12 mb-24 luxury-border"
              >
                <h4 className="text-3xl font-display font-black mb-12 tracking-tighter text-white">{editingPlan ? 'Modify Node' : 'Provision New Node'}</h4>
                <form onSubmit={handleSubmit} className="grid grid-cols-1 md:grid-cols-2 gap-10">
                  <div className="space-y-4">
                    <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Node Designation</label>
                    <input 
                      type="text" 
                      placeholder="e.g. Starter Node"
                      className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium"
                      value={formData.name}
                      onChange={e => setFormData({...formData, name: e.target.value})}
                      required
                    />
                  </div>
                  <div className="space-y-4">
                    <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Credits (INR)</label>
                    <input 
                      type="text" 
                      placeholder="e.g. 499"
                      className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium"
                      value={formData.price}
                      onChange={e => setFormData({...formData, price: e.target.value})}
                      required
                    />
                  </div>
                  <div className="space-y-4">
                    <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Billing Cycle</label>
                    <input 
                      type="text" 
                      placeholder="/mo"
                      className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium"
                      value={formData.period}
                      onChange={e => setFormData({...formData, period: e.target.value})}
                      required
                    />
                  </div>
                  <div className="space-y-4">
                    <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Infrastructure Category</label>
                    <select 
                      className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium appearance-none"
                      value={formData.category}
                      onChange={e => setFormData({...formData, category: e.target.value as 'minecraft' | 'vps'})}
                      required
                    >
                      <option value="minecraft" className="bg-black">Minecraft Hosting</option>
                      <option value="vps" className="bg-black">VPS Hosting</option>
                    </select>
                  </div>
                  <div className="space-y-4">
                    <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Tier / Sub-Category</label>
                    <select 
                      className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium appearance-none"
                      value={formData.subCategory}
                      onChange={e => setFormData({...formData, subCategory: e.target.value as 'premium' | 'budget'})}
                      required
                    >
                      <option value="premium" className="bg-black">Premium / Enterprise</option>
                      <option value="budget" className="bg-black">Budget / Standard</option>
                    </select>
                  </div>
                  <div className="space-y-4">
                    <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Display Order</label>
                    <input 
                      type="number" 
                      className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium"
                      value={formData.order}
                      onChange={e => setFormData({...formData, order: Number(e.target.value)})}
                      required
                    />
                  </div>
                  <div className="space-y-4 md:col-span-2">
                    <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">
                      {formData.category === 'vps' ? 'VPS Specifications (RAM, CPU, Storage, Bandwidth...)' : 'Minecraft Features (comma separated)'}
                    </label>
                    <textarea 
                      placeholder={formData.category === 'vps' ? "8GB DDR4 RAM, 4 vCores EPYC, 100GB NVMe, 10Gbps Network..." : "Unlimited Slots, Pterodactyl Panel, DDoS Protection..."}
                      className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium min-h-[120px]"
                      value={formData.features}
                      onChange={e => setFormData({...formData, features: e.target.value})}
                      required
                    />
                  </div>
                  <div className="md:col-span-2">
                    <label className="flex items-center gap-4 cursor-pointer group">
                      <div className={cn(
                        "w-8 h-8 rounded-lg border border-white/10 flex items-center justify-center transition-all",
                        formData.popular ? "bg-brand-primary border-brand-primary" : "bg-white/[0.02] group-hover:border-white/20"
                      )}>
                        {formData.popular && <Check className="w-5 h-5 text-white" />}
                      </div>
                      <input 
                        type="checkbox" 
                        className="hidden"
                        checked={formData.popular}
                        onChange={e => setFormData({...formData, popular: e.target.checked})}
                      />
                      <span className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-400 group-hover:text-white transition-colors">Mark as Most Popular</span>
                    </label>
                  </div>
                  <div className="md:col-span-2 pt-6">
                    <button type="submit" className="w-full py-5 rounded-2xl premium-gradient text-white font-black text-[10px] uppercase tracking-[0.4em] shadow-2xl shadow-brand-primary/40">
                      {editingPlan ? 'Commit Changes' : 'Provision Infrastructure'}
                    </button>
                  </div>
                </form>
              </motion.div>
            )}

            <div className="grid grid-cols-1 gap-6 mb-24">
              {filteredAdminPlans.map(plan => (
                <div key={plan.id} className="glass-card p-8 flex flex-col md:flex-row items-center justify-between gap-8 luxury-border group">
                  <div className="flex items-center gap-10">
                    <div className="w-20 h-20 bg-white/[0.02] border border-white/5 rounded-3xl flex items-center justify-center font-black text-brand-primary text-2xl shadow-inner group-hover:scale-110 transition-transform duration-500">
                      #{plan.order}
                    </div>
                    <div>
                      <div className="flex items-center gap-4 mb-2">
                        <span className="px-3 py-1 rounded-full bg-brand-primary/10 text-brand-primary text-[8px] font-black uppercase tracking-[0.3em] border border-brand-primary/20">{plan.category || 'minecraft'}</span>
                        <span className="px-3 py-1 rounded-full bg-white/5 text-slate-400 text-[8px] font-black uppercase tracking-[0.3em] border border-white/10">{plan.subCategory || 'premium'}</span>
                        <h4 className="font-display font-black text-3xl tracking-tighter text-white">{plan.name}</h4>
                      </div>
                      <p className="text-slate-500 font-bold tracking-widest text-[12px] uppercase">₹{plan.price}{plan.period}</p>
                    </div>
                  </div>
                  <div className="flex items-center gap-6">
                    <button 
                      onClick={() => {
                        setEditingPlan(plan);
                        setFormData({
                          name: plan.name,
                          price: plan.price,
                          period: plan.period,
                          features: plan.features.join(', '),
                          popular: plan.popular,
                          order: plan.order,
                          category: plan.category || 'minecraft',
                          subCategory: plan.subCategory || 'premium'
                        });
                        setIsAdding(false);
                        window.scrollTo({ top: 0, behavior: 'smooth' });
                      }}
                      className="w-14 h-14 rounded-2xl bg-white/[0.03] border border-white/5 flex items-center justify-center text-slate-500 hover:text-brand-primary hover:border-brand-primary/30 hover:bg-white/[0.08] transition-all"
                    >
                      <Edit2 className="w-6 h-6" />
                    </button>
                    <button 
                      onClick={() => handleDelete(plan.id)}
                      className="w-14 h-14 rounded-2xl bg-white/[0.03] border border-white/5 flex items-center justify-center text-slate-500 hover:text-red-500 hover:border-red-500/30 hover:bg-white/[0.08] transition-all"
                    >
                      <Trash2 className="w-6 h-6" />
                    </button>
                  </div>
                </div>
              ))}
            </div>

            <div className="glass-card p-12 luxury-border">
              <h4 className="text-3xl font-display font-black mb-8 tracking-tighter text-white">System Actions</h4>
              <div className="flex flex-wrap gap-6">
                <button 
                  onClick={seedMinecraftPlans}
                  className="px-8 py-4 rounded-xl bg-white/[0.03] border border-white/5 text-[10px] font-black uppercase tracking-[0.3em] text-slate-400 hover:text-white hover:bg-white/[0.08] transition-all"
                >
                  Seed MC Premium
                </button>
                <button 
                  onClick={seedMinecraftBudgetPlans}
                  className="px-8 py-4 rounded-xl bg-white/[0.03] border border-white/5 text-[10px] font-black uppercase tracking-[0.3em] text-slate-400 hover:text-white hover:bg-white/[0.08] transition-all"
                >
                  Seed MC Budget
                </button>
                <button 
                  onClick={seedVPSPlans}
                  className="px-8 py-4 rounded-xl bg-white/[0.03] border border-white/5 text-[10px] font-black uppercase tracking-[0.3em] text-slate-400 hover:text-white hover:bg-white/[0.08] transition-all"
                >
                  Seed VPS Plans
                </button>
              </div>
            </div>
          </>
        ) : (
          <motion.div 
            initial={{ opacity: 0, y: 20 }}
            animate={{ opacity: 1, y: 0 }}
            className="glass-card p-12 luxury-border"
          >
            <h4 className="text-3xl font-display font-black mb-12 tracking-tighter text-white">Announcement Banner Settings</h4>
            <form onSubmit={handleBannerSubmit} className="grid grid-cols-1 md:grid-cols-2 gap-10">
              <div className="space-y-4 md:col-span-2">
                <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Banner Message</label>
                <input 
                  type="text" 
                  className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium"
                  value={bannerForm.text}
                  onChange={e => setBannerForm({...bannerForm, text: e.target.value})}
                  required
                />
              </div>
              <div className="space-y-4">
                <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Coupon Code</label>
                <input 
                  type="text" 
                  className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium"
                  value={bannerForm.code}
                  onChange={e => setBannerForm({...bannerForm, code: e.target.value})}
                  required
                />
              </div>
              <div className="space-y-4">
                <label className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-500 ml-1">Discount Amount</label>
                <input 
                  type="text" 
                  className="w-full bg-white/[0.02] border border-white/5 rounded-2xl px-6 py-4 text-white focus:outline-none focus:border-brand-primary/50 transition-all font-medium"
                  value={bannerForm.discount}
                  onChange={e => setBannerForm({...bannerForm, discount: e.target.value})}
                  required
                />
              </div>
              <div className="md:col-span-2">
                <label className="flex items-center gap-4 cursor-pointer group">
                  <div className={cn(
                    "w-8 h-8 rounded-lg border border-white/10 flex items-center justify-center transition-all",
                    bannerForm.active ? "bg-brand-primary border-brand-primary" : "bg-white/[0.02] group-hover:border-white/20"
                  )}>
                    {bannerForm.active && <Check className="w-5 h-5 text-white" />}
                  </div>
                  <input 
                    type="checkbox" 
                    className="hidden"
                    checked={bannerForm.active}
                    onChange={e => setBannerForm({...bannerForm, active: e.target.checked})}
                  />
                  <span className="text-[10px] font-black uppercase tracking-[0.3em] text-slate-400 group-hover:text-white transition-colors">Enable Banner</span>
                </label>
              </div>
              <div className="md:col-span-2 flex justify-end">
                <button type="submit" className="btn-premium premium-gradient px-16 py-5 rounded-2xl font-black text-[10px] uppercase tracking-[0.4em] shadow-2xl">
                  Save Settings
                </button>
              </div>
            </form>
          </motion.div>
        )}
      </div>
    </section>
  );
};

const Footer = () => {
  return (
    <footer className="bg-black border-t border-white/5 pt-32 pb-16 relative noise-bg texture-3d">
      <div className="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
        <div className="grid grid-cols-1 md:grid-cols-4 gap-16 mb-24">
          <div className="col-span-1 md:col-span-2">
            <div className="flex items-center gap-3 mb-8">
              <div className="w-12 h-12 premium-gradient rounded-2xl flex items-center justify-center shadow-2xl shadow-brand-primary/40">
                <Server className="text-white w-7 h-7" />
              </div>
              <span className="text-3xl font-display font-black tracking-tighter text-white">Rexa<span className="text-brand-primary">Cloud</span></span>
            </div>
            <p className="text-slate-500 max-w-sm mb-10 text-lg leading-relaxed font-medium tracking-tight">
              Providing premium cloud hosting solutions for the next generation of digital creators and enterprise businesses.
            </p>
            <div className="flex gap-4">
              <a href={DISCORD_LINK} target="_blank" rel="noopener noreferrer" className="w-14 h-14 glass-card flex items-center justify-center hover:bg-brand-primary/10 hover:border-brand-primary/30 transition-all text-slate-400 hover:text-brand-primary luxury-border">
                <MessageSquare className="w-6 h-6" />
              </a>
            </div>
          </div>
          
          <div>
            <h4 className="font-display font-black mb-8 text-white tracking-widest uppercase text-[10px]">Quick Links</h4>
            <ul className="space-y-5 text-slate-500 font-medium tracking-tight">
              <li><a href="#" className="hover:text-brand-primary transition-colors">Home</a></li>
              <li><a href="#features" className="hover:text-brand-primary transition-colors">Features</a></li>
              <li><a href="#plans" className="hover:text-brand-primary transition-colors">Plans</a></li>
              <li><a href={DISCORD_LINK} className="hover:text-brand-primary transition-colors">Support</a></li>
            </ul>
          </div>
        </div>
        
        <div className="pt-12 border-t border-white/5 text-center text-slate-600 text-[10px] font-black tracking-[0.5em] uppercase">
          <p>© {new Date().getFullYear()} Rexa Cloud. All rights reserved.</p>
        </div>
      </div>
    </footer>
  );
};

export default function App() {
  const [plans, setPlans] = useState<Plan[]>([]);
  const [user, setUser] = useState<User | null>(null);
  const [isAdmin, setIsAdmin] = useState(false);
  const [loading, setLoading] = useState(true);
  const [showAdmin, setShowAdmin] = useState(false);
  const [bannerSettings, setBannerSettings] = useState<BannerSettings | null>(null);

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      const cards = document.querySelectorAll('.glass-card');
      cards.forEach((card) => {
        const rect = (card as HTMLElement).getBoundingClientRect();
        const x = e.clientX - rect.left;
        const y = e.clientY - rect.top;
        (card as HTMLElement).style.setProperty('--mouse-x', `${x}px`);
        (card as HTMLElement).style.setProperty('--mouse-y', `${y}px`);
      });
    };

    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);

  useEffect(() => {
    // Test connection
    const testConnection = async () => {
      try {
        await getDocFromServer(doc(db, 'test', 'connection'));
      } catch (error) {
        if(error instanceof Error && error.message.includes('the client is offline')) {
          console.error("Please check your Firebase configuration.");
        }
      }
    };
    testConnection();

    // Listen for banner settings
    const unsubscribeSettings = onSnapshot(doc(db, 'settings', 'banner'), (doc) => {
      if (doc.exists()) {
        setBannerSettings(doc.data() as BannerSettings);
      }
    });

    // Listen for plans
    const q = query(collection(db, 'plans'), orderBy('order', 'asc'));
    const unsubscribePlans = onSnapshot(q, (snapshot) => {
      const plansData = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      })) as Plan[];
      setPlans(plansData);
      setLoading(false);

      // Auto-seed if empty (only for the first time)
      if (plansData.length === 0) {
        const defaultPlans = [
          // Minecraft Hosting (Premium)
          { name: 'Grass Tier', price: '199', period: '/mo', features: ['2GB Dedicated RAM', '2 vCores AMD EPYC', '20GB NVMe Storage', 'Unlimited Slots', 'Pterodactyl Panel', 'DDoS Protection'], popular: false, order: 1, category: 'minecraft', subCategory: 'premium' },
          { name: 'Iron Tier', price: '399', period: '/mo', features: ['4GB Dedicated RAM', '4 vCores AMD EPYC', '40GB NVMe Storage', 'Unlimited Slots', 'Pterodactyl Panel', 'DDoS Protection'], popular: true, order: 2, category: 'minecraft', subCategory: 'premium' },
          { name: 'Diamond Tier', price: '799', period: '/mo', features: ['8GB Dedicated RAM', '6 vCores AMD EPYC', '80GB NVMe Storage', 'Unlimited Slots', 'Pterodactyl Panel', 'DDoS Protection'], popular: false, order: 3, category: 'minecraft', subCategory: 'premium' },
          { name: 'Netherite Tier', price: '1499', period: '/mo', features: ['16GB Dedicated RAM', '8 vCores AMD EPYC', '160GB NVMe Storage', 'Unlimited Slots', 'Pterodactyl Panel', 'DDoS Protection'], popular: false, order: 4, category: 'minecraft', subCategory: 'premium' }
        ];
        defaultPlans.forEach(p => addDoc(collection(db, 'plans'), p));
      }
    }, (error) => {
      console.error("Firestore Error (plans):", error);
    });

    // Listen for auth
    const unsubscribeAuth = onAuthStateChanged(auth, async (currentUser) => {
      setUser(currentUser);
      if (currentUser) {
        // Check if user is admin
        const userDoc = await getDoc(doc(db, 'users', currentUser.uid));
        if (userDoc.exists()) {
          setIsAdmin(userDoc.data().role === 'admin');
        } else {
          // Create user doc if not exists
          const isDefaultAdmin = currentUser.email === "ghoshsima874@gmail.com";
          const userData = {
            uid: currentUser.uid,
            email: currentUser.email,
            displayName: currentUser.displayName,
            role: isDefaultAdmin ? 'admin' : 'user'
          };
          await setDoc(doc(db, 'users', currentUser.uid), userData);
          setIsAdmin(isDefaultAdmin);
        }
      } else {
        setIsAdmin(false);
        setShowAdmin(false);
      }
    });

    return () => {
      unsubscribeSettings();
      unsubscribePlans();
      unsubscribeAuth();
    };
  }, []);

  return (
    <div className="min-h-screen selection:bg-brand-primary/30">
      <Banner settings={bannerSettings} />
      <Navbar 
        user={user} 
        isAdmin={isAdmin} 
        onAdminToggle={() => setShowAdmin(!showAdmin)} 
        showAdmin={showAdmin} 
      />
      <main>
        {showAdmin && isAdmin ? (
          <AdminDashboard plans={plans} isAdmin={isAdmin} bannerSettings={bannerSettings} />
        ) : (
          <>
            <Hero />
            <TrustedBy />
            <Features />
            <WhyChooseUs />
            <Gallery />
            {loading ? (
              <div className="py-24 text-center">
                <div className="w-12 h-12 border-4 border-brand-primary border-t-transparent rounded-full animate-spin mx-auto mb-4" />
                <p className="text-slate-400">Loading plans...</p>
              </div>
            ) : (
              <Plans plans={plans} />
            )}
            <Footer />
          </>
        )}
      </main>
    </div>
  );
}

