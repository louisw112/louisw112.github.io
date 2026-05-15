<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1, user-scalable=no">
<meta name="theme-color" content="#1F4E79">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="apple-mobile-web-app-title" content="Tri-Tracker">
<title>Tri-Tracker · Olympic 30.08.2026</title>
<link rel="manifest" href="data:application/json,{"name":"Tri-Tracker","short_name":"Tri-Tracker","display":"standalone","background_color":"%231F4E79","theme_color":"%231F4E79","start_url":"./","icons":[]}">
<link rel="apple-touch-icon" href="data:image/svg+xml,%3Csvg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22%3E%3Crect width=%22100%22 height=%22100%22 fill=%22%231F4E79%22/%3E%3Ctext x=%2250%22 y=%2275%22 font-size=%2280%22 text-anchor=%22middle%22%3E%F0%9F%8F%8A%E2%80%8D%E2%99%82%EF%B8%8F%3C/text%3E%3C/svg%3E">
<script src="https://cdn.tailwindcss.com"></script>
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script src="https://unpkg.com/prop-types@15.8.1/prop-types.min.js"></script>
<script src="https://unpkg.com/recharts/umd/Recharts.min.js"></script>
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
<style>
  body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif; -webkit-tap-highlight-color: transparent; }
  #loading { position: fixed; inset: 0; background: linear-gradient(135deg, #1F4E79, #2563eb); color: white; display: flex; flex-direction: column; align-items: center; justify-content: center; font-family: -apple-system, sans-serif; }
  #loading.hidden { display: none; }
</style>
</head>
<body>
<div id="loading">
  <div style="font-size: 48px; margin-bottom: 16px;">🏊‍♂️🚴‍♂️🏃‍♂️</div>
  <div style="font-size: 18px; font-weight: bold;">Tri-Tracker lädt...</div>
  <div style="font-size: 12px; opacity: 0.7; margin-top: 8px;">Olympic · 30.08.2026</div>
</div>
<div id="root"></div>
<script type="text/babel" data-presets="react">

// React + Recharts globals
const { useState, useEffect, useMemo } = React;
const { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, ResponsiveContainer, ReferenceLine } = Recharts;

// === ICON COMPONENTS (inline SVG, replaces lucide-react) ===
const makeIcon = (svgInner) => ({size=18, className=''}) => (
  <svg width={size} height={size} className={className} viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" dangerouslySetInnerHTML={{__html: svgInner}}/>
);
const Activity = makeIcon('<polyline points="22 12 18 12 15 21 9 3 6 12 2 12"/>');
const Calendar = makeIcon('<rect x="3" y="4" width="18" height="18" rx="2" ry="2"/><line x1="16" y1="2" x2="16" y2="6"/><line x1="8" y1="2" x2="8" y2="6"/><line x1="3" y1="10" x2="21" y2="10"/>');
const TrendingUp = makeIcon('<polyline points="22 7 13.5 15.5 8.5 10.5 2 17"/><polyline points="16 7 22 7 22 13"/>');
const Dumbbell = makeIcon('<path d="m6.5 6.5 11 11"/><path d="m21 21-1-1"/><path d="m3 3 1 1"/><path d="m18 22 4-4"/><path d="m2 6 4-4"/><path d="m3 10 7-7"/><path d="m14 21 7-7"/>');
const Waves = makeIcon('<path d="M2 6c.6.5 1.2 1 2.5 1C7 7 7 5 9.5 5c2.6 0 2.4 2 5 2 2.5 0 2.5-2 5-2 1.3 0 1.9.5 2.5 1"/><path d="M2 12c.6.5 1.2 1 2.5 1 2.5 0 2.5-2 5-2 2.6 0 2.4 2 5 2 2.5 0 2.5-2 5-2 1.3 0 1.9.5 2.5 1"/><path d="M2 18c.6.5 1.2 1 2.5 1 2.5 0 2.5-2 5-2 2.6 0 2.4 2 5 2 2.5 0 2.5-2 5-2 1.3 0 1.9.5 2.5 1"/>');
const Scale = makeIcon('<path d="m16 16 3-8 3 8c-.87.65-1.92 1-3 1s-2.13-.35-3-1Z"/><path d="m2 16 3-8 3 8c-.87.65-1.92 1-3 1s-2.13-.35-3-1Z"/><path d="M7 21h10"/><path d="M12 3v18"/><path d="M3 7h2c2 0 5-1 7-2 2 1 5 2 7 2h2"/>');
const FileText = makeIcon('<path d="M14.5 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V7.5L14.5 2z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/>');
const Download = makeIcon('<path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/>');
const AlertTriangle = makeIcon('<path d="m21.73 18-8-14a2 2 0 0 0-3.48 0l-8 14A2 2 0 0 0 4 21h16a2 2 0 0 0 1.73-3Z"/><line x1="12" y1="9" x2="12" y2="13"/><line x1="12" y1="17" x2="12.01" y2="17"/>');
const CheckCircle = makeIcon('<path d="M22 11.08V12a10 10 0 1 1-5.93-9.14"/><polyline points="22 4 12 14.01 9 11.01"/>');
const Target = makeIcon('<circle cx="12" cy="12" r="10"/><circle cx="12" cy="12" r="6"/><circle cx="12" cy="12" r="2"/>');
const Home = makeIcon('<path d="m3 9 9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/>');
const BookOpen = makeIcon('<path d="M2 3h6a4 4 0 0 1 4 4v14a3 3 0 0 0-3-3H2z"/><path d="M22 3h-6a4 4 0 0 0-4 4v14a3 3 0 0 1 3-3h7z"/>');
const Apple = makeIcon('<path d="M12 20.94c1.5 0 2.75 1.06 4 1.06 3 0 6-8 6-12.22A4.91 4.91 0 0 0 17 5c-2.22 0-4 1.44-5 2-1-.56-2.78-2-5-2a4.9 4.9 0 0 0-5 4.78C2 14 5 22 8 22c1.25 0 2.5-1.06 4-1.06Z"/><path d="M10 2c1 .5 2 2 2 5"/>');
const Timer = makeIcon('<line x1="10" y1="2" x2="14" y2="2"/><line x1="12" y1="14" x2="15" y2="11"/><circle cx="12" cy="14" r="8"/>');
const ChevronRight = makeIcon('<polyline points="9 18 15 12 9 6"/>');
const ChevronDown = makeIcon('<polyline points="6 9 12 15 18 9"/>');
const ChevronLeft = makeIcon('<polyline points="15 18 9 12 15 6"/>');
const Menu = makeIcon('<line x1="3" y1="12" x2="21" y2="12"/><line x1="3" y1="6" x2="21" y2="6"/><line x1="3" y1="18" x2="21" y2="18"/>');
const User = makeIcon('<path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/>');

// ============ PLAN DATA ============
const PLAN_START = new Date('2026-05-12');
const RACE_DATE = new Date('2026-08-30');

const PHASES = [
  { num: 1, name: 'Phase 1: Grundlage', start: '2026-05-12', end: '2026-06-08', color: '#2E75B6',
    description: 'Schwimm-Technik etablieren, aerobe Basis aufbauen, Bewegungsqualität. Volumen ~6,5–7,5 h/Woche (Plus-Version). Wichtigster Output: saubere Kraul-Technik. Investiere jetzt in Coaching.' },
  { num: 2, name: 'Phase 2: Aufbau', start: '2026-06-09', end: '2026-07-06', color: '#1F4E79',
    description: 'Durchgehende Schwimm-Distanzen (Ziel 750 m). Erste Schwellenintervalle. Erste Brick-Workouts. Volumen ~8–9 h/Woche. Test Woche 8: Schaffst du 750 m am Stück?' },
  { num: 3, name: 'Phase 3: Spezifisch', start: '2026-07-07', end: '2026-08-03', color: '#5B9BD5',
    description: 'Freiwasserschwimmen, Renn-Pace, längere Bricks. Mini-Wettkampfsimulation. Volumen ~9,5–12 h/Woche (Peak Woche 11). Beckenschwimmen ≠ Freiwasser!' },
  { num: 4, name: 'Phase 4: Peak & Taper', start: '2026-08-04', end: '2026-08-30', color: '#9DC3E6',
    description: 'Letzte harte Einheit Woche 13, dann progressives Tapern. Schärfe halten, Müdigkeit abbauen. Race-Woche 16: Carb-Load, viel trinken, früh schlafen. Kein neues Equipment!' },
];

const PLAN_WEEKS = [
  { num: 1, range: '12.05. – 18.05.', hours: 7.0, phase: 1,
    focus: 'Plus-Start. Schwimm-Technik, aerobe Basis, funktionelle Kraft. Fettverlust-Modus.',
    days: [
      { d: 'Di', date: '12.05.', type: 'hart', title: 'Triple-Session', dur: '~2 h',
        details: ['Schwimmen 30 min: Technik (16×25 m mit langem Gleiten)', 'Kraft A funktionell: Kniebeuge 4×5-6 · Bankdrücken 3×5-6 · RDL 3×6-8 · Klimmzug 3×6-10 · Plank 3×45s', 'Lauf 25 min Zone 2 abends, ≥4h nach Kraft'] },
      { d: 'Mi', date: '13.05.', type: 'rest', title: 'Aktive Erholung', dur: '~30 min',
        details: ['Walk 30 min (NEAT für Fettabbau)', 'Mobility 10 min optional', 'Schlaf ≥8 h'] },
      { d: 'Do', date: '14.05.', type: 'moderat', title: 'Schwimmen + Bike', dur: '~1 h',
        details: ['Schwimmen 30 min: 16×25 m Kraul · Atmung beidseitig üben', 'Rad 35 min Zone 2 locker (kann auch Bike-Commute sein)'] },
      { d: 'Fr', date: '15.05.', type: 'moderat', title: 'Rad lang', dur: '60 min',
        details: ['Rad 60 min Zone 2 · gleichmäßige Trittfrequenz 80-90'] },
      { d: 'Sa', date: '16.05.', type: 'hart', title: 'Kraft + Lauf', dur: '~1,5 h',
        details: ['Kraft B funktionell: Hip Thrust 3×6-8 · Schulterdrücken 3×6-8 · Bulgarian SS 3×8/B · Rudern 3×6-8 · Beinheben 3×10', 'Lauf 40 min Z2 (≥2 h nach Kraft)'] },
      { d: 'So', date: '17.05.', type: 'hart', title: 'Rad lang + Schwimmen', dur: '~1,75 h',
        details: ['Rad 75 min Z2 · Verpflegung üben', 'Schwimmen 30 min Technik: 8×50 m, beidseitig atmen'] },
      { d: 'Mo', date: '18.05.', type: 'moderat', title: 'Lauf locker', dur: '30 min',
        details: ['Lauf 30 min Z2 sehr locker · ggf. Steigerung am Ende 3×20s'] },
    ]},
  { num: 2, range: '19.05. – 25.05.', hours: 7.5, phase: 1,
    focus: 'Konstanz aufbauen. Schwimm-Frequenz hoch halten. NEAT-Volumen erhöhen.',
    days: [
      { d: 'Di', date: '19.05.', type: 'moderat', title: 'Schwimmen + Kraft A', dur: '~1,5 h',
        details: ['Schwimmen 35 min: 4×100 m oder 12×50 m', 'Kraft A (evtl. +2,5 kg auf Kniebeuge/Bankdrücken)'] },
      { d: 'Mi', date: '20.05.', type: 'moderat', title: 'Lauf mit Steigerungen', dur: '35 min',
        details: ['Lauf 35 min Z2 · letzte 10 min: 4×30 s Steigerung mit 90 s Trab'] },
      { d: 'Do', date: '21.05.', type: 'moderat', title: 'Schwimmen + Bike', dur: '~1,25 h',
        details: ['Schwimmen 40 min: 6×50 m + Pull-Buoy-Übungen', 'Rad 35 min Z2 (Commute möglich)'] },
      { d: 'Fr', date: '22.05.', type: 'rest', title: 'Aktive Erholung', dur: '30 min',
        details: ['Walk 30 min · Mobility 10 min'] },
      { d: 'Sa', date: '23.05.', type: 'hart', title: 'Rad lang', dur: '80 min',
        details: ['Rad 80 min Z2 · längste Ausfahrt bisher · Trinken alle 15 min'] },
      { d: 'So', date: '24.05.', type: 'hart', title: 'Schwimmen + Lauf', dur: '~1,75 h',
        details: ['Schwimmen 40 min Technik · 8×50 m bestmögliche Form', 'Lauf 45 min Z2 (≥2 h nach Schwimmen)'] },
      { d: 'Mo', date: '25.05.', type: 'moderat', title: 'Kraft B', dur: '40 min',
        details: ['Kraft B (Progression: 1-2 Wdh. mehr oder +2,5 kg)'] },
    ]},
  { num: 3, range: '26.05. – 01.06.', hours: 7.5, phase: 1,
    focus: 'Erste Schwimm-Distanzen über 50 m. Erste Rad-Intensität. Lauf-Volumen steigt.',
    days: [
      { d: 'Di', date: '26.05.', type: 'moderat', title: 'Schwimmen + Kraft A', dur: '~1,5 h',
        details: ['Schwimmen 40 min: 8×50 m + 2×100 m Versuch', 'Kraft A - progressive Steigerung'] },
      { d: 'Mi', date: '27.05.', type: 'hart', title: 'Rad Intervalle', dur: '60 min',
        details: ['Rad 60 min: 15 min Z2 + 5×(30 s hart / 90 s locker) + Rest Z2'] },
      { d: 'Do', date: '28.05.', type: 'hart', title: 'Schwimmen + Lauf', dur: '~1,5 h',
        details: ['Schwimmen 45 min: 8×50 m + 2×100 m + beidseitig atmen', 'Lauf 35 min Z2'] },
      { d: 'Fr', date: '29.05.', type: 'rest', title: 'Erholung', dur: '30 min',
        details: ['Walk 30 min oder Yoga 20 min'] },
      { d: 'Sa', date: '30.05.', type: 'hart', title: 'Rad lang', dur: '90 min',
        details: ['Rad 90 min Z2 · Verpflegung üben · längste bisher'] },
      { d: 'So', date: '31.05.', type: 'hart', title: 'Schwimmen + Lauf', dur: '~1,75 h',
        details: ['Schwimmen 45 min: 4×100 m kontinuierlich versuchen', 'Lauf 45 min Z2'] },
      { d: 'Mo', date: '01.06.', type: 'moderat', title: 'Kraft B', dur: '40 min', details: ['Kraft B - Progression'] },
    ]},
  { num: 4, range: '02.06. – 08.06.', hours: 5.0, phase: 1,
    focus: 'DELOAD-Woche. Volumen -30%. Technik festigen. Recovery für Phase 2.',
    days: [
      { d: 'Di', date: '02.06.', type: 'moderat', title: 'Schwimm-Test + Kraft leicht', dur: '~1 h',
        details: ['Schwimmen 30 min: TEST - wie weit am Stück? Notieren!', 'Kraft A - 70% Gewicht, gleiche Wdh.'] },
      { d: 'Mi', date: '03.06.', type: 'rest', title: 'Walk', dur: '30 min', details: ['Walk 30 min'] },
      { d: 'Do', date: '04.06.', type: 'moderat', title: 'Schwimmen + Lauf', dur: '~1 h',
        details: ['Schwimmen 35 min Technik', 'Lauf 30 min Z2 sehr locker'] },
      { d: 'Fr', date: '05.06.', type: 'moderat', title: 'Rad', dur: '50 min', details: ['Rad 50 min Z2 locker'] },
      { d: 'Sa', date: '06.06.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Komplette Pause'] },
      { d: 'So', date: '07.06.', type: 'hart', title: 'Rad + Schwimmen', dur: '~1,5 h',
        details: ['Rad 70 min Z2 locker', 'Schwimmen 30 min Technik locker'] },
      { d: 'Mo', date: '08.06.', type: 'moderat', title: 'Lauf', dur: '30 min', details: ['Lauf 30 min Z2 entspannt'] },
    ]},
  { num: 5, range: '09.06. – 15.06.', hours: 8.5, phase: 2,
    focus: 'Phase 2 startet. Erste durchgehende Schwimm-Distanzen (Ziel 200 m). Längere Long-Days.',
    days: [
      { d: 'Di', date: '09.06.', type: 'moderat', title: 'Schwimmen + Kraft A', dur: '~1,5 h',
        details: ['Schwimmen 50 min: 200 m am Stück versuchen · sonst 4×100 m', 'Kraft A'] },
      { d: 'Mi', date: '10.06.', type: 'hart', title: 'Rad Tempo', dur: '75 min',
        details: ['Rad 75 min: 15 min Z2 + 5×4 min Tempo (Z3) mit 2 min Z2 + Rest Z2'] },
      { d: 'Do', date: '11.06.', type: 'hart', title: 'Schwimmen + Lauf', dur: '~1,5 h',
        details: ['Schwimmen 50 min: 6×100 m + Technik', 'Lauf 40 min Z2'] },
      { d: 'Fr', date: '12.06.', type: 'rest', title: 'Erholung', dur: '30 min', details: ['Walk 30 min'] },
      { d: 'Sa', date: '13.06.', type: 'hart', title: 'Rad lang', dur: '90 min',
        details: ['Rad 90 min Z2 · Verpflegung üben (Banane, Riegel, 750ml)'] },
      { d: 'So', date: '14.06.', type: 'hart', title: 'Lauf lang + Schwimmen', dur: '~1,75 h',
        details: ['Lauf 50 min Z2', 'Schwimmen 45 min: 6×50 m + 4×25 m Sprint-Technik'] },
      { d: 'Mo', date: '15.06.', type: 'moderat', title: 'Kraft B', dur: '40 min', details: ['Kraft B'] },
    ]},
  { num: 6, range: '16.06. – 22.06.', hours: 9.0, phase: 2,
    focus: 'Erstes Brick-Training. Lauf-Intervalle starten. Schwimmziel 300 m am Stück.',
    days: [
      { d: 'Di', date: '16.06.', type: 'moderat', title: 'Schwimmen + Kraft A', dur: '~1,5 h',
        details: ['Schwimmen 55 min: 300 m am Stück + 6×50 m', 'Kraft A'] },
      { d: 'Mi', date: '17.06.', type: 'hart', title: 'Lauf Intervalle', dur: '45 min',
        details: ['Lauf 45 min: 10 min ein + 8×(1 min Z4 / 90 s Trab) + 10 min aus'] },
      { d: 'Do', date: '18.06.', type: 'moderat', title: 'Schwimmen + Bike', dur: '~1,25 h',
        details: ['Schwimmen 50 min Technik · 8×100 m', 'Rad 35 min Z2 locker'] },
      { d: 'Fr', date: '19.06.', type: 'rest', title: 'Erholung', dur: '30 min', details: ['Walk + Mobility'] },
      { d: 'Sa', date: '20.06.', type: 'hart', title: 'Rad Tempo', dur: '90 min',
        details: ['Rad 90 min: 20 min ein + 5×5 min Tempo Z3 mit 3 min Z2 + Rest Z2'] },
      { d: 'So', date: '21.06.', type: 'hart', title: 'ERSTER BRICK + Schwimmen', dur: '~2,25 h',
        details: ['BRICK: Rad 60 min Z2 → sofort umziehen → Lauf 20 min Z2 (Beine fühlen sich seltsam an, normal!)', 'Schwimmen 45 min Technik (mit Abstand)'] },
      { d: 'Mo', date: '22.06.', type: 'hart', title: 'Kraft + Lauf', dur: '~1,5 h',
        details: ['Kraft B', 'Lauf 30 min Z2 locker'] },
    ]},
  { num: 7, range: '23.06. – 29.06.', hours: 9.5, phase: 2,
    focus: 'Schwimmziel 500 m. Schwellenarbeit Rad. Brick wird Routine.',
    days: [
      { d: 'Di', date: '23.06.', type: 'moderat', title: 'Schwimmen + Kraft A', dur: '~1,5 h',
        details: ['Schwimmen 55 min: 500 m am Stück · sonst 2×300 m', 'Kraft A'] },
      { d: 'Mi', date: '24.06.', type: 'hart', title: 'Rad Schwelle', dur: '80 min',
        details: ['Rad 80 min: 15 min ein + 5×5 min Z4 mit 2 min Z2 + 10 min aus'] },
      { d: 'Do', date: '25.06.', type: 'hart', title: 'Schwimmen + Lauf', dur: '~1,75 h',
        details: ['Schwimmen 55 min: 4×200 m + Technik', 'Lauf 45 min Z2'] },
      { d: 'Fr', date: '26.06.', type: 'rest', title: 'Erholung', dur: '30 min', details: ['Walk 30 min'] },
      { d: 'Sa', date: '27.06.', type: 'hart', title: 'BRICK + Schwimmen', dur: '~2,5 h',
        details: ['BRICK: Rad 75 min Z2 → Lauf 25 min Z2', 'Schwimmen 35 min Technik (mit Abstand)'] },
      { d: 'So', date: '28.06.', type: 'hart', title: 'Lauf lang + Schwimmen', dur: '~2,25 h',
        details: ['Lauf 60 min Z2 - längster Lauf bisher', 'Schwimmen 50 min Technik'] },
      { d: 'Mo', date: '29.06.', type: 'moderat', title: 'Kraft B', dur: '40 min', details: ['Kraft B'] },
    ]},
  { num: 8, range: '30.06. – 06.07.', hours: 6.0, phase: 2,
    focus: 'DELOAD. Volumen -35%. Test am Ende: 750 m am Stück?',
    days: [
      { d: 'Di', date: '30.06.', type: 'moderat', title: 'Schwimmen + Kraft leicht', dur: '~1,25 h',
        details: ['Schwimmen 45 min: Technik + 4×200 m', 'Kraft A leicht (70%)'] },
      { d: 'Mi', date: '01.07.', type: 'moderat', title: 'Rad locker', dur: '60 min', details: ['Rad 60 min Z2 locker'] },
      { d: 'Do', date: '02.07.', type: 'moderat', title: 'Schwimmen + Lauf', dur: '~1,25 h',
        details: ['Schwimmen 45 min: 2×300 m locker', 'Lauf 35 min Z2'] },
      { d: 'Fr', date: '03.07.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Pause'] },
      { d: 'Sa', date: '04.07.', type: 'moderat', title: 'BRICK leicht', dur: '75 min',
        details: ['Rad 60 min Z2 → Lauf 15 min Z2'] },
      { d: 'So', date: '05.07.', type: 'hart', title: 'SCHWIMM-TEST + Lauf', dur: '~1,5 h',
        details: ['Schwimmen 55 min: TEST - wie weit am Stück? Ziel 750 m', 'Lauf 40 min Z2 locker'] },
      { d: 'Mo', date: '06.07.', type: 'moderat', title: 'Kraft B leicht', dur: '40 min', details: ['Kraft B leicht'] },
    ]},
  { num: 9, range: '07.07. – 13.07.', hours: 10.0, phase: 3,
    focus: 'Phase 3. Erstes Freiwasser-Schwimmen. Längere Bricks. Schwellen-Intervalle Lauf.',
    days: [
      { d: 'Di', date: '07.07.', type: 'hart', title: 'Schwimmen + Kraft A', dur: '~2 h',
        details: ['Schwimmen 65 min: 750 m am Stück + 4×100 m race-pace', 'Kraft A'] },
      { d: 'Mi', date: '08.07.', type: 'hart', title: 'Lauf Schwelle', dur: '55 min',
        details: ['Lauf 55 min: 10 min ein + 8×(2 min Z4 / 2 min Trab) + 10 min aus'] },
      { d: 'Do', date: '09.07.', type: 'hart', title: 'Schwimmen + Rad', dur: '~1,75 h',
        details: ['Schwimmen 60 min: 2×400 m + Technik', 'Rad 45 min Z2'] },
      { d: 'Fr', date: '10.07.', type: 'rest', title: 'Erholung', dur: '30 min', details: ['Walk + Mobility'] },
      { d: 'Sa', date: '11.07.', type: 'hart', title: 'FREIWASSER + Rad', dur: '~2 h',
        details: ['FREIWASSER 40 min mit Neopren · Orientierung üben · NIE alleine!', 'Rad 75 min Z2'] },
      { d: 'So', date: '12.07.', type: 'hart', title: 'BRICK lang', dur: '~2 h',
        details: ['BRICK: Rad 90 min Z2 → Lauf 30 min Z3 (knapp Renn-Pace)'] },
      { d: 'Mo', date: '13.07.', type: 'hart', title: 'Kraft + Lauf', dur: '~1,25 h',
        details: ['Kraft B', 'Lauf 30 min Z2 locker'] },
    ]},
  { num: 10, range: '14.07. – 20.07.', hours: 11.0, phase: 3,
    focus: 'Schwimmziel 1000 m. Längere Rad-Schwellen. Freiwasser-Distanz steigern.',
    days: [
      { d: 'Di', date: '14.07.', type: 'hart', title: 'Schwimmen + Kraft A', dur: '~2 h',
        details: ['Schwimmen 70 min: 1000 m am Stück versuchen', 'Kraft A'] },
      { d: 'Mi', date: '15.07.', type: 'hart', title: 'Rad Schwelle', dur: '90 min',
        details: ['Rad 90 min: 15 min ein + 5×6 min Z4 mit 3 min Z2 + 15 min aus'] },
      { d: 'Do', date: '16.07.', type: 'hart', title: 'Schwimmen + Lauf', dur: '~2 h',
        details: ['Schwimmen 65 min: 3×400 m race-pace', 'Lauf 50 min Z2'] },
      { d: 'Fr', date: '17.07.', type: 'rest', title: 'Erholung', dur: '30 min', details: ['Walk · Yoga'] },
      { d: 'Sa', date: '18.07.', type: 'hart', title: 'FREIWASSER + Rad lang', dur: '~2,75 h',
        details: ['FREIWASSER 45 min · Distanz steigern · gerade Linien üben', 'Rad 100 min Z2'] },
      { d: 'So', date: '19.07.', type: 'hart', title: 'BRICK + Schwimmen', dur: '~2,5 h',
        details: ['BRICK: Rad 75 min Z2 → Lauf 40 min Z2', 'Schwimmen 45 min locker'] },
      { d: 'Mo', date: '20.07.', type: 'moderat', title: 'Kraft B', dur: '40 min', details: ['Kraft B - letzte schwere in Phase 3'] },
    ]},
  { num: 11, range: '21.07. – 27.07.', hours: 12.0, phase: 3,
    focus: 'PEAK-WOCHE. Höchstes Volumen. Mini-Tri-Simulation. Schwimm-Ziel 1500 m.',
    days: [
      { d: 'Di', date: '21.07.', type: 'hart', title: 'Schwimmen + Kraft leicht', dur: '~2,5 h',
        details: ['Schwimmen 80 min: 1500 m am Stück (= Renn-Distanz!)', 'Kraft A leicht - Erhaltung'] },
      { d: 'Mi', date: '22.07.', type: 'hart', title: 'Lauf race-pace', dur: '65 min',
        details: ['Lauf 65 min: 15 min ein + 6×3 min Renn-Pace (Z4) mit 2 min Trab + 15 min aus'] },
      { d: 'Do', date: '23.07.', type: 'hart', title: 'Schwimmen + Rad', dur: '~2 h',
        details: ['Schwimmen 70 min: 4×200 m Renn-Pace + Technik', 'Rad 50 min Z2'] },
      { d: 'Fr', date: '24.07.', type: 'rest', title: 'Erholung', dur: '—', details: ['Pause · Vorbereitung Mini-Tri Sa'] },
      { d: 'Sa', date: '25.07.', type: 'hart', title: 'MINI-TRI SIMULATION', dur: '~2,75 h',
        details: ['Freiwasser 30 min → Wechsel → Rad 100 min Z2 → Wechsel → Lauf 25 min Z2 · Wechselzonen üben!'] },
      { d: 'So', date: '26.07.', type: 'hart', title: 'Rad sehr lang', dur: '2,5 h',
        details: ['Rad 2,5 h Z2 · längste Ausfahrt der Vorbereitung'] },
      { d: 'Mo', date: '27.07.', type: 'hart', title: 'Kraft + Lauf', dur: '~1,5 h',
        details: ['Kraft B leicht', 'Lauf 45 min Z2 sehr locker'] },
    ]},
  { num: 12, range: '28.07. – 03.08.', hours: 7.5, phase: 3,
    focus: 'DELOAD nach Peak. Volumen -35%. Erholung dominiert.',
    days: [
      { d: 'Di', date: '28.07.', type: 'moderat', title: 'Schwimmen + Kraft leicht', dur: '~1,5 h',
        details: ['Schwimmen 55 min locker · 4×200 m', 'Kraft A leicht (60%)'] },
      { d: 'Mi', date: '29.07.', type: 'moderat', title: 'Rad', dur: '65 min', details: ['Rad 65 min Z2'] },
      { d: 'Do', date: '30.07.', type: 'moderat', title: 'Schwimmen + Lauf', dur: '~1,5 h',
        details: ['Schwimmen 55 min Technik', 'Lauf 45 min Z2'] },
      { d: 'Fr', date: '31.07.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Pause'] },
      { d: 'Sa', date: '01.08.', type: 'hart', title: 'FREIWASSER + Rad', dur: '~2 h',
        details: ['Freiwasser 30 min · Renn-Tempo', 'Rad 80 min Z2'] },
      { d: 'So', date: '02.08.', type: 'moderat', title: 'BRICK', dur: '95 min',
        details: ['BRICK: Rad 65 min Z2 → Lauf 30 min Z2'] },
      { d: 'Mo', date: '03.08.', type: 'moderat', title: 'Kraft B leicht', dur: '40 min', details: ['Kraft B leicht'] },
    ]},
  { num: 13, range: '04.08. – 10.08.', hours: 9.0, phase: 4,
    focus: 'Letzte harte Woche. Renn-Pace schärfen. Wettkampf-Simulation.',
    days: [
      { d: 'Di', date: '04.08.', type: 'hart', title: 'Schwimmen + Kraft A', dur: '~2 h',
        details: ['Schwimmen 75 min: 4×300 m Renn-Pace', 'Kraft A - letzte schwere Einheit'] },
      { d: 'Mi', date: '05.08.', type: 'hart', title: 'Rad race-pace', dur: '80 min',
        details: ['Rad 80 min: 15 min ein + 3×12 min Renn-Pace (Z3/Z4) mit 5 min Z2 + 10 min aus'] },
      { d: 'Do', date: '06.08.', type: 'hart', title: 'Schwimmen + Lauf race-pace', dur: '~2 h',
        details: ['Schwimmen 65 min: 6×150 m race-pace', 'Lauf 55 min: 10 min ein + 4×1 km Renn-Pace mit 2 min Trab + 10 min aus'] },
      { d: 'Fr', date: '07.08.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Pause · Material vorbereiten'] },
      { d: 'Sa', date: '08.08.', type: 'hart', title: 'WETTKAMPF-SIMULATION', dur: '~2,5 h',
        details: ['Olympic simulieren: Freiwasser 1200 m → Rad 35 km → Lauf 8 km · Renn-Verpflegung testen'] },
      { d: 'So', date: '09.08.', type: 'moderat', title: 'Locker erholen', dur: '45 min',
        details: ['Rad 45 min Z1/Z2 sehr locker oder Walk'] },
      { d: 'Mo', date: '10.08.', type: 'moderat', title: 'Kraft B leicht', dur: '35 min', details: ['Kraft B leicht - LETZTE Krafteinheit'] },
    ]},
  { num: 14, range: '11.08. – 17.08.', hours: 5.5, phase: 4,
    focus: 'TAPER startet. Volumen -40%. Intensität bleibt kurz und scharf.',
    days: [
      { d: 'Di', date: '11.08.', type: 'moderat', title: 'Schwimmen', dur: '50 min',
        details: ['Schwimmen 50 min: Technik + 4×100 m race-pace'] },
      { d: 'Mi', date: '12.08.', type: 'moderat', title: 'Rad', dur: '60 min',
        details: ['Rad 60 min: 3×5 min Renn-Pace mit 5 min Z2'] },
      { d: 'Do', date: '13.08.', type: 'moderat', title: 'Schwimmen + Lauf', dur: '~1,5 h',
        details: ['Schwimmen 45 min locker', 'Lauf 35 min: 3×500 m Renn-Pace mit Trabpause'] },
      { d: 'Fr', date: '14.08.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Pause'] },
      { d: 'Sa', date: '15.08.', type: 'moderat', title: 'BRICK leicht', dur: '75 min',
        details: ['Rad 60 min Z2 → Lauf 15 min Z2'] },
      { d: 'So', date: '16.08.', type: 'moderat', title: 'FREIWASSER + Lauf', dur: '~1,25 h',
        details: ['Freiwasser 45 min Renn-Tempo · letzte längere OWS', 'Lauf 25 min Z2 locker'] },
      { d: 'Mo', date: '17.08.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Pause'] },
    ]},
  { num: 15, range: '18.08. – 24.08.', hours: 4.0, phase: 4,
    focus: 'TIEFER TAPER. Volumen -50%. Schärfe halten mit kurzen Beschleunigungen.',
    days: [
      { d: 'Di', date: '18.08.', type: 'moderat', title: 'Schwimmen kurz', dur: '40 min',
        details: ['Schwimmen 40 min: Technik + 6×50 m Sprint-Steigerung'] },
      { d: 'Mi', date: '19.08.', type: 'moderat', title: 'Rad kurz', dur: '45 min',
        details: ['Rad 45 min Z2 + 4×30 s hart mit 90 s locker'] },
      { d: 'Do', date: '20.08.', type: 'moderat', title: 'Schwimmen + Lauf', dur: '~1 h',
        details: ['Schwimmen 30 min locker', 'Lauf 25 min Z2 + 3×30 s Steigerung'] },
      { d: 'Fr', date: '21.08.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Pause'] },
      { d: 'Sa', date: '22.08.', type: 'moderat', title: 'BRICK kurz', dur: '50 min',
        details: ['Rad 40 min Z2 → Lauf 10 min Z2 · Wechselzone üben'] },
      { d: 'So', date: '23.08.', type: 'moderat', title: 'Schwimmen leicht', dur: '~1 h',
        details: ['Schwimmen 30 min sehr locker · Walk 30 min'] },
      { d: 'Mo', date: '24.08.', type: 'rest', title: 'Ruhetag', dur: '—', details: ['Pause'] },
    ]},
  { num: 16, range: '25.08. – 30.08.', hours: 2.5, phase: 4,
    focus: 'RACE-WOCHE! Minimal-Volumen. Carb-Load Do/Fr. Frühes Schlafen.',
    days: [
      { d: 'Di', date: '25.08.', type: 'moderat', title: 'Schwimmen sehr leicht', dur: '30 min',
        details: ['Schwimmen 30 min: 200 m kontrolliert + Technik · KEINE Intensität'] },
      { d: 'Mi', date: '26.08.', type: 'moderat', title: 'Rad + Lauf kurz', dur: '~45 min',
        details: ['Rad 30 min Z2 + 2×30 s hart', 'Lauf 15 min Z2 + 2×30 s Steigerung'] },
      { d: 'Do', date: '27.08.', type: 'moderat', title: 'Schwimmen + Lauf kurz', dur: '35 min',
        details: ['Schwimmen 20 min locker', 'Lauf 15 min Z2 + 3×20 s Steigerung'] },
      { d: 'Fr', date: '28.08.', type: 'rest', title: 'Ruhetag', dur: '—',
        details: ['PAUSE. Hydration ++. Carb-Load (8-10 g/kg Carbs = ~600-800 g). Material checken. Frühes Bett.'] },
      { d: 'Sa', date: '29.08.', type: 'moderat', title: 'Aktivierung', dur: '35 min',
        details: ['Rad 15 min Z2 + 1×30 s hart', 'Lauf 10 min Z2 + 2 Steigerungen', 'Schwimmen 10 min locker', 'Strecke besichtigen, früh schlafen'] },
      { d: 'So', date: '30.08.', type: 'race', title: '🏆 WETTKAMPF: OLYMPIC TRI', dur: '~3 h',
        details: ['1,5 km Schwimmen → 40 km Rad → 10 km Laufen', 'Aufwärmen: 10 min Joggen + 5 min Schwimmen + Steigerungen', 'DU HAST DAS!'] },
    ]},
];

// Build lookup map for quick access
const PLAN_BY_DATE = {};
PLAN_WEEKS.forEach(w => w.days.forEach(d => {
  const fullDate = `2026-${d.date.split('.')[1]}-${d.date.split('.')[0]}`;
  PLAN_BY_DATE[fullDate] = { ...d, week: w.num, phase: w.phase, focus: w.focus };
}));

// ============ MACROS ============
const PHASE_MACROS = {
  1: { hart: { kcal: 3300, p: 170, f: 75, c: 490 }, moderat: { kcal: 2700, p: 170, f: 70, c: 350 }, rest: { kcal: 2300, p: 170, f: 70, c: 250 } },
  2: { hart: { kcal: 3500, p: 170, f: 75, c: 540 }, moderat: { kcal: 2850, p: 170, f: 70, c: 390 }, rest: { kcal: 2350, p: 170, f: 70, c: 260 } },
  3: { hart: { kcal: 3800, p: 170, f: 75, c: 615 }, moderat: { kcal: 3050, p: 170, f: 70, c: 440 }, rest: { kcal: 2400, p: 170, f: 70, c: 275 } },
  4: { hart: { kcal: 3500, p: 170, f: 75, c: 540 }, moderat: { kcal: 2850, p: 170, f: 70, c: 390 }, rest: { kcal: 2350, p: 170, f: 70, c: 260 } },
};

// ============ HELPERS ============
const todayISO = () => new Date().toISOString().slice(0, 10);
const daysBetween = (d1, d2) => Math.floor((new Date(d2) - new Date(d1)) / (1000 * 60 * 60 * 24));
const formatDate = (iso) => new Date(iso).toLocaleDateString('de-DE', { weekday: 'short', day: '2-digit', month: '2-digit', year: 'numeric' });
const formatDateShort = (iso) => new Date(iso).toLocaleDateString('de-DE', { day: '2-digit', month: '2-digit' });

const getCurrentPhase = (date = new Date()) => {
  for (const p of PHASES) {
    if (date >= new Date(p.start) && date <= new Date(p.end + 'T23:59:59')) return p;
  }
  if (date < new Date(PHASES[0].start)) return { name: 'Vor Plan-Start', color: '#888', num: 0 };
  return { name: 'Nach Wettkampf', color: '#888', num: 4 };
};

const getDaysToRace = () => Math.max(0, daysBetween(todayISO(), '2026-08-30'));
const getTodaysPlan = () => PLAN_BY_DATE[todayISO()] || null;
const getCurrentWeek = () => {
  const t = new Date();
  for (let i = 0; i < PLAN_WEEKS.length; i++) {
    const w = PLAN_WEEKS[i];
    const firstDay = w.days[0];
    const lastDay = w.days[w.days.length - 1];
    const firstISO = `2026-${firstDay.date.split('.')[1]}-${firstDay.date.split('.')[0]}`;
    const lastISO = `2026-${lastDay.date.split('.')[1]}-${lastDay.date.split('.')[0]}`;
    if (t >= new Date(firstISO) && t <= new Date(lastISO + 'T23:59:59')) return i + 1;
  }
  return 1;
};

// ============ STORAGE ============
const useStorage = (key, initial) => {
  const [value, setValue] = useState(() => {
    try {
      const stored = localStorage.getItem(key);
      return stored ? JSON.parse(stored) : initial;
    } catch (e) { return initial; }
  });
  const save = (newVal) => {
    setValue(newVal);
    try { localStorage.setItem(key, JSON.stringify(newVal)); } catch (e) { console.error(e); }
  };
  return [value, save];
};

// ============ UI ============
const TypeBadge = ({ type }) => {
  const styles = {
    hart: 'bg-red-100 text-red-800 border-red-300',
    moderat: 'bg-yellow-100 text-yellow-800 border-yellow-300',
    rest: 'bg-gray-100 text-gray-700 border-gray-300',
    race: 'bg-purple-100 text-purple-800 border-purple-300',
  };
  const labels = { hart: 'HART', moderat: 'MODERAT', rest: 'RUHE', race: 'WETTKAMPF' };
  return <span className={`inline-block px-2 py-0.5 text-[10px] font-bold rounded border ${styles[type] || styles.moderat}`}>{labels[type] || type.toUpperCase()}</span>;
};

const Card = ({ children, className = '' }) => (
  <div className={`bg-white rounded-2xl border border-slate-200 p-4 shadow-sm ${className}`}>{children}</div>
);

const SectionTitle = ({ icon: Icon, children, action }) => (
  <div className="flex items-center justify-between mb-3">
    <div className="flex items-center gap-2 text-slate-800">
      {Icon && <Icon size={18} className="text-blue-700" />}
      <h2 className="text-base font-bold">{children}</h2>
    </div>
    {action}
  </div>
);

// ============ DASHBOARD ============
const DashboardTab = ({ weightLog, swimLog, strengthLog, dailyLogs, goTo }) => {
  const plan = getTodaysPlan();
  const phase = getCurrentPhase();
  const phaseNum = phase.num || 1;
  const daysLeft = getDaysToRace();
  const week = getCurrentWeek();
  const macros = plan ? PHASE_MACROS[phaseNum][plan.type === 'race' ? 'hart' : plan.type] : null;

  const latestWeight = weightLog.length > 0 ? weightLog[weightLog.length - 1] : null;
  const startWeight = 78.2;
  const weightChange = latestWeight ? (latestWeight.weight - startWeight).toFixed(1) : null;
  const longestSwim = swimLog.reduce((max, s) => Math.max(max, s.longestContinuous || 0), 0);

  const recommendations = useMemo(() => {
    const recs = [];
    const recent7 = dailyLogs.filter(l => daysBetween(l.date, todayISO()) <= 7);
    const missed = recent7.filter(l => l.status === 'missed' || (l.status === undefined && l.completed === false)).length;
    const less = recent7.filter(l => l.status === 'less').length;
    const more = recent7.filter(l => l.status === 'more').length;
    if (missed >= 3) recs.push({ type: 'warning', title: `${missed} verpasste Einheiten in 7 Tagen`, text: 'Reduziere die Intensität nächste Woche um 20%. Schlaf checken, Stress checken.' });
    if (less >= 3) recs.push({ type: 'warning', title: `${less}× weniger geschafft als geplant`, text: 'Dein Körper signalisiert Überlastung. Plan-Volumen zu hoch? Sleep, Ernährung checken.' });
    if (more >= 3) recs.push({ type: 'info', title: `${more}× mehr gemacht als geplant`, text: 'Aufpassen — wiederholtes Überziehen führt zu Burnout. Vertrau dem Plan.' });

    const rpes = recent7.filter(l => l.rpe).map(l => l.rpe);
    if (rpes.length >= 4) {
      const avgRPE = rpes.reduce((a, b) => a + b, 0) / rpes.length;
      if (avgRPE > 7.5) recs.push({ type: 'warning', title: `Ø RPE ${avgRPE.toFixed(1)} — sehr hoch`, text: 'Konstant harte Sessions = Übertraining. Ein paar bewusst lockerer machen.' });
    }

    const sleeps = recent7.filter(l => l.sleep).map(l => l.sleep);
    if (sleeps.length >= 3) {
      const avg = sleeps.reduce((a, b) => a + b, 0) / sleeps.length;
      if (avg < 7) recs.push({ type: 'warning', title: `Schlaf-Schnitt: ${avg.toFixed(1)} h`, text: 'Unter 7 h ist Recovery-Killer. Priorität: 8 h, früh ins Bett.' });
    }

    const sores = recent7.filter(l => l.soreness).map(l => l.soreness);
    if (sores.length >= 3) {
      const avgSore = sores.reduce((a, b) => a + b, 0) / sores.length;
      if (avgSore >= 7) recs.push({ type: 'warning', title: `Müdigkeit hoch (Ø ${avgSore.toFixed(1)}/10)`, text: 'Erholung greift nicht. Ruhetag einschieben, Massage, mehr Schlaf, evtl. Eisen checken.' });
    }

    const hrs = recent7.filter(l => l.restingHR).map(l => l.restingHR).slice(-7);
    if (hrs.length >= 5) {
      const baseline = Math.min(...hrs);
      const latest = hrs[hrs.length - 1];
      if (latest - baseline >= 7) recs.push({ type: 'warning', title: `Ruhepuls erhöht (${latest} vs Basis ${baseline} bpm)`, text: '+7 bpm = klares Recovery-Signal. Heute lockerer trainieren oder Pause.' });
    }

    if (weightLog.length >= 3) {
      const r3 = weightLog.slice(-3);
      const span = daysBetween(r3[0].date, r3[2].date) / 7;
      if (span > 0) {
        const rate = (r3[2].weight - r3[0].weight) / span;
        if (rate < -0.7) recs.push({ type: 'warning', title: `Gewicht fällt zu schnell (${rate.toFixed(2)} kg/Wo)`, text: 'Zu schnell = Muskelverlust + Performance-Einbruch. +250 kcal/Tag.' });
        else if (rate > 0.3) recs.push({ type: 'info', title: `Gewicht steigt (+${rate.toFixed(2)} kg/Wo)`, text: 'Falls nicht gewollt: 200 kcal/Tag weniger an Ruhetagen.' });
      }
    }
    if (phaseNum >= 2 && longestSwim < 200) recs.push({ type: 'warning', title: 'Schwimm-Volumen hängt hinterher', text: 'In Phase 2 solltest du Richtung 300 m am Stück arbeiten. Schwimm-Coaching?' });
    if (recs.length === 0) recs.push({ type: 'success', title: 'Alles im grünen Bereich', text: 'Keine Auffälligkeiten. Konsistenz ist der Hebel.' });
    return recs;
  }, [weightLog, dailyLogs, longestSwim, phaseNum]);

  return (
    <div className="space-y-3">
      <Card className="bg-gradient-to-br from-blue-700 to-blue-900 text-white border-0">
        <div className="flex items-center justify-between">
          <div>
            <div className="text-blue-200 text-xs uppercase tracking-wide">Tage bis Wettkampf</div>
            <div className="text-5xl font-bold mt-1">{daysLeft}</div>
          </div>
          <div className="text-right">
            <div className="text-blue-200 text-xs">30. Aug 2026</div>
            <div className="text-blue-100 text-sm font-medium mt-1">Woche {week}/16</div>
            <div className="text-blue-200 text-xs mt-0.5">{phase.name}</div>
          </div>
        </div>
      </Card>

      <Card>
        <SectionTitle icon={Calendar} action={
          <button onClick={() => goTo('plan')} className="text-blue-700 text-xs font-medium flex items-center gap-0.5">
            Plan ansehen <ChevronRight size={14} />
          </button>
        }>Heute · {formatDate(todayISO())}</SectionTitle>
        {plan ? (
          <>
            <div className="flex items-center gap-2 mb-2">
              <TypeBadge type={plan.type} />
              <span className="text-xs text-slate-500">Woche {plan.week}</span>
            </div>
            <div className="font-semibold text-slate-800 mb-2">{plan.title}</div>
            {plan.details.length > 0 && (
              <ul className="text-sm text-slate-600 space-y-1">
                {plan.details.map((s, i) => <li key={i}>• {s}</li>)}
              </ul>
            )}
            {macros && (
              <div className="mt-3 pt-3 border-t border-slate-200">
                <div className="text-xs text-slate-500 uppercase tracking-wide mb-1.5">Makro-Ziele heute</div>
                <div className="grid grid-cols-4 gap-1.5 text-center">
                  <div className="bg-slate-50 rounded-lg p-1.5">
                    <div className="text-[10px] text-slate-500">kcal</div>
                    <div className="font-bold text-slate-800 text-sm">{macros.kcal}</div>
                  </div>
                  <div className="bg-slate-50 rounded-lg p-1.5">
                    <div className="text-[10px] text-slate-500">P</div>
                    <div className="font-bold text-slate-800 text-sm">{macros.p}g</div>
                  </div>
                  <div className="bg-slate-50 rounded-lg p-1.5">
                    <div className="text-[10px] text-slate-500">F</div>
                    <div className="font-bold text-slate-800 text-sm">{macros.f}g</div>
                  </div>
                  <div className="bg-slate-50 rounded-lg p-1.5">
                    <div className="text-[10px] text-slate-500">C</div>
                    <div className="font-bold text-slate-800 text-sm">{macros.c}g</div>
                  </div>
                </div>
              </div>
            )}
          </>
        ) : (
          <div className="text-slate-500 text-sm">Kein Plan für heute. Genieß die Pause.</div>
        )}
      </Card>

      <Card>
        <SectionTitle icon={Target}>Coach-Hinweise</SectionTitle>
        <div className="space-y-2">
          {recommendations.map((r, i) => (
            <div key={i} className={`flex gap-2 p-2.5 rounded-lg border ${
              r.type === 'warning' ? 'bg-amber-50 border-amber-200' :
              r.type === 'success' ? 'bg-emerald-50 border-emerald-200' :
              'bg-blue-50 border-blue-200'
            }`}>
              {r.type === 'warning' ? <AlertTriangle size={16} className="text-amber-600 flex-shrink-0 mt-0.5" /> :
               r.type === 'success' ? <CheckCircle size={16} className="text-emerald-600 flex-shrink-0 mt-0.5" /> :
               <Activity size={16} className="text-blue-600 flex-shrink-0 mt-0.5" />}
              <div>
                <div className="font-semibold text-sm text-slate-800">{r.title}</div>
                <div className="text-xs text-slate-600 mt-0.5">{r.text}</div>
              </div>
            </div>
          ))}
        </div>
      </Card>

      <div className="grid grid-cols-2 gap-2">
        <button onClick={() => goTo('tracker')} className="bg-white rounded-2xl border border-slate-200 p-3 text-left">
          <div className="text-[10px] text-slate-500 uppercase tracking-wide">Gewicht</div>
          <div className="text-xl font-bold text-slate-800 mt-1">{latestWeight ? `${latestWeight.weight} kg` : '—'}</div>
          {weightChange !== null && (
            <div className={`text-xs mt-0.5 ${weightChange > 0 ? 'text-amber-600' : weightChange < 0 ? 'text-emerald-600' : 'text-slate-500'}`}>
              {weightChange > 0 ? '+' : ''}{weightChange} kg
            </div>
          )}
        </button>
        <button onClick={() => goTo('tracker')} className="bg-white rounded-2xl border border-slate-200 p-3 text-left">
          <div className="text-[10px] text-slate-500 uppercase tracking-wide">Längstes Kraul</div>
          <div className="text-xl font-bold text-slate-800 mt-1">{longestSwim > 0 ? `${longestSwim} m` : '—'}</div>
          <div className="text-xs text-slate-500 mt-0.5">Ziel: 1.500 m</div>
        </button>
      </div>
    </div>
  );
};

// ============ PLAN TAB ============
const PlanTab = () => {
  const currentWeek = getCurrentWeek();
  const [viewWeek, setViewWeek] = useState(currentWeek);
  const [expandedDay, setExpandedDay] = useState(todayISO());
  const week = PLAN_WEEKS[viewWeek - 1];
  const phaseInfo = PHASES[week.phase - 1];

  return (
    <div className="space-y-3">
      <Card>
        <div className="flex items-center justify-between mb-3">
          <button onClick={() => setViewWeek(Math.max(1, viewWeek - 1))} disabled={viewWeek === 1}
            className="p-1.5 rounded-lg bg-slate-100 disabled:opacity-30">
            <ChevronLeft size={18} />
          </button>
          <div className="text-center">
            <div className="text-xs text-slate-500">Woche {viewWeek}/16 {viewWeek === currentWeek && '· AKTUELL'}</div>
            <div className="font-bold text-slate-800">{week.range}</div>
            <div className="text-xs text-slate-500 mt-0.5">~{week.hours} h Trainings­volumen</div>
          </div>
          <button onClick={() => setViewWeek(Math.min(16, viewWeek + 1))} disabled={viewWeek === 16}
            className="p-1.5 rounded-lg bg-slate-100 disabled:opacity-30">
            <ChevronRight size={18} />
          </button>
        </div>
        <div className="px-3 py-2 rounded-lg" style={{ backgroundColor: phaseInfo.color + '20', borderLeft: `4px solid ${phaseInfo.color}` }}>
          <div className="text-xs font-semibold" style={{ color: phaseInfo.color }}>{phaseInfo.name}</div>
          <div className="text-sm text-slate-700 mt-1">{week.focus}</div>
        </div>
      </Card>

      <div className="space-y-2">
        {week.days.map((day, i) => {
          const fullDate = `2026-${day.date.split('.')[1]}-${day.date.split('.')[0]}`;
          const isToday = fullDate === todayISO();
          const isExpanded = expandedDay === fullDate;
          return (
            <Card key={i} className={`${isToday ? 'ring-2 ring-blue-500' : ''}`}>
              <button onClick={() => setExpandedDay(isExpanded ? null : fullDate)} className="w-full text-left">
                <div className="flex items-center justify-between">
                  <div className="flex items-center gap-3">
                    <div className="w-10 text-center">
                      <div className="text-xs font-bold text-slate-500">{day.d}</div>
                      <div className="text-sm font-semibold text-slate-700">{day.date}</div>
                    </div>
                    <div className="flex-1">
                      <div className="flex items-center gap-2 mb-0.5">
                        <TypeBadge type={day.type} />
                        {isToday && <span className="text-[10px] font-bold text-blue-700 bg-blue-100 px-1.5 py-0.5 rounded">HEUTE</span>}
                      </div>
                      <div className="font-semibold text-slate-800 text-sm">{day.title}</div>
                      <div className="text-xs text-slate-500">{day.dur}</div>
                    </div>
                  </div>
                  {isExpanded ? <ChevronDown size={18} className="text-slate-400" /> : <ChevronRight size={18} className="text-slate-400" />}
                </div>
              </button>
              {isExpanded && day.details.length > 0 && (
                <ul className="mt-3 pt-3 border-t border-slate-100 space-y-1.5">
                  {day.details.map((det, j) => <li key={j} className="text-sm text-slate-700">• {det}</li>)}
                </ul>
              )}
            </Card>
          );
        })}
      </div>
    </div>
  );
};

// ============ TRACKER TAB (with sub-tabs) ============
const TrackerTab = ({ weightLog, setWeightLog, swimLog, setSwimLog, strengthLog, setStrengthLog, dailyLogs, setDailyLogs, cardioTests, setCardioTests, nutritionLogs, setNutritionLogs, mobilityLogs, setMobilityLogs }) => {
  const [sub, setSub] = useState('log');
  const subs = [
    { id: 'log', label: 'Tageslog' },
    { id: 'weight', label: 'Gewicht' },
    { id: 'strength', label: 'Kraft' },
    { id: 'swim', label: 'Schwimm' },
    { id: 'cardio', label: 'Tests' },
    { id: 'nutrition', label: 'Ernährung' },
    { id: 'mobility', label: 'Mobility' },
  ];

  return (
    <div className="space-y-3">
      <div className="flex gap-1 overflow-x-auto pb-1 -mx-1 px-1">
        {subs.map(s => (
          <button key={s.id} onClick={() => setSub(s.id)}
            className={`px-3 py-1.5 rounded-full text-xs font-semibold whitespace-nowrap ${sub === s.id ? 'bg-blue-700 text-white' : 'bg-white text-slate-600 border border-slate-200'}`}>
            {s.label}
          </button>
        ))}
      </div>
      {sub === 'log' && <DailyLogSub dailyLogs={dailyLogs} setDailyLogs={setDailyLogs} />}
      {sub === 'weight' && <WeightSub weightLog={weightLog} setWeightLog={setWeightLog} />}
      {sub === 'strength' && <StrengthSub strengthLog={strengthLog} setStrengthLog={setStrengthLog} />}
      {sub === 'swim' && <SwimSub swimLog={swimLog} setSwimLog={setSwimLog} />}
      {sub === 'cardio' && <CardioSub cardioTests={cardioTests} setCardioTests={setCardioTests} />}
      {sub === 'nutrition' && <NutritionSub nutritionLogs={nutritionLogs} setNutritionLogs={setNutritionLogs} />}
      {sub === 'mobility' && <MobilitySub mobilityLogs={mobilityLogs} setMobilityLogs={setMobilityLogs} />}
    </div>
  );
};

const DailyLogSub = ({ dailyLogs, setDailyLogs }) => {
  const [date, setDate] = useState(todayISO());
  const [status, setStatus] = useState('planned');
  const [actualDuration, setActualDuration] = useState('');
  const [rpe, setRpe] = useState(6);
  const [energy, setEnergy] = useState(7);
  const [sleep, setSleep] = useState(8);
  const [restingHR, setRestingHR] = useState('');
  const [soreness, setSoreness] = useState(3);
  const [notes, setNotes] = useState('');
  const plan = PLAN_BY_DATE[date];

  // Load existing entry when date changes
  useEffect(() => {
    const existing = dailyLogs.find(l => l.date === date);
    if (existing) {
      setStatus(existing.status || (existing.completed === false ? 'missed' : 'planned'));
      setActualDuration(existing.actualDuration || '');
      setRpe(existing.rpe || 6);
      setEnergy(existing.energy || 7);
      setSleep(existing.sleep || 8);
      setRestingHR(existing.restingHR || '');
      setSoreness(existing.soreness || 3);
      setNotes(existing.notes || '');
    } else {
      setStatus('planned'); setActualDuration(''); setRpe(6); setEnergy(7);
      setSleep(8); setRestingHR(''); setSoreness(3); setNotes('');
    }
  }, [date]);

  const submit = async () => {
    const log = {
      date,
      completed: status !== 'missed',
      status,
      actualDuration: actualDuration ? parseInt(actualDuration) : null,
      rpe: status !== 'missed' ? parseInt(rpe) : null,
      restingHR: restingHR ? parseInt(restingHR) : null,
      soreness: parseInt(soreness),
      energy: parseInt(energy),
      sleep: parseFloat(sleep),
      notes,
      planned: plan?.title || ''
    };
    const filtered = dailyLogs.filter(l => l.date !== date);
    await setDailyLogs([...filtered, log].sort((a, b) => b.date.localeCompare(a.date)));
    alert('Gespeichert!');
  };

  const statusBtn = (val, label, icon) => {
    const active = status === val;
    const colors = {
      planned: active ? 'bg-emerald-100 border-emerald-300 text-emerald-800' : 'bg-white border-slate-300 text-slate-600',
      more:    active ? 'bg-blue-100 border-blue-300 text-blue-800' : 'bg-white border-slate-300 text-slate-600',
      less:    active ? 'bg-amber-100 border-amber-300 text-amber-800' : 'bg-white border-slate-300 text-slate-600',
      missed:  active ? 'bg-red-100 border-red-300 text-red-800' : 'bg-white border-slate-300 text-slate-600',
    };
    return (
      <button onClick={() => setStatus(val)}
        className={`flex flex-col items-center justify-center py-2 px-1 rounded-lg border text-xs font-medium ${colors[val]}`}>
        <span className="text-base font-bold">{icon}</span>
        <span className="mt-0.5">{label}</span>
      </button>
    );
  };

  const rpeLabel = (v) => v <= 2 ? 'Sehr leicht' : v <= 4 ? 'Leicht' : v <= 6 ? 'Moderat' : v <= 8 ? 'Hart' : 'Maximum';
  const rpeColor = (v) => v <= 3 ? 'bg-emerald-500' : v <= 5 ? 'bg-blue-500' : v <= 7 ? 'bg-yellow-500' : v <= 9 ? 'bg-orange-500' : 'bg-red-500';
  const soreColor = (v) => v <= 3 ? 'bg-emerald-500' : v <= 6 ? 'bg-yellow-500' : v <= 8 ? 'bg-orange-500' : 'bg-red-500';

  return (
    <>
      <Card>
        <SectionTitle icon={Activity}>Tages-Log</SectionTitle>
        <div className="space-y-3">
          <input type="date" value={date} onChange={e => setDate(e.target.value)}
            className="w-full px-3 py-2 border border-slate-300 rounded-lg" />
          {plan && (
            <div className="bg-slate-50 rounded-lg p-2">
              <div className="flex items-center gap-2 mb-0.5">
                <TypeBadge type={plan.type} />
                <span className="text-xs text-slate-500">geplant · {plan.dur}</span>
              </div>
              <div className="text-sm font-medium text-slate-800">{plan.title}</div>
            </div>
          )}

          <div>
            <label className="text-xs text-slate-600 mb-1.5 block font-semibold">Wie lief's?</label>
            <div className="grid grid-cols-2 gap-2">
              {statusBtn('planned', 'Wie geplant', '✓')}
              {statusBtn('more', 'Mehr gemacht', '+')}
              {statusBtn('less', 'Weniger geschafft', '−')}
              {statusBtn('missed', 'Verpasst', '✗')}
            </div>
          </div>

          {(status === 'more' || status === 'less') && (
            <div className="bg-slate-50 rounded-lg p-2.5">
              <label className="text-xs text-slate-600">Tatsächliche Dauer (min)</label>
              <input type="number" value={actualDuration} onChange={e => setActualDuration(e.target.value)}
                placeholder="z.B. 35"
                className="w-full mt-1 px-3 py-2 border border-slate-300 rounded-lg" />
              {status === 'more' && <div className="text-[10px] text-amber-600 mt-1">⚠️ Wiederholt mehr machen = Übertraining-Risiko</div>}
            </div>
          )}

          {status !== 'missed' && (
            <div>
              <div className="flex items-center justify-between mb-1.5">
                <label className="text-xs text-slate-600 font-semibold">RPE — wie hart? (1–10)</label>
                <div className="flex items-center gap-2">
                  <span className={`px-2 py-0.5 rounded text-xs font-bold text-white ${rpeColor(rpe)}`}>{rpe}</span>
                  <span className="text-xs text-slate-600">{rpeLabel(rpe)}</span>
                </div>
              </div>
              <input type="range" min="1" max="10" value={rpe} onChange={e => setRpe(parseInt(e.target.value))}
                className="w-full accent-blue-600" />
            </div>
          )}

          <div className="grid grid-cols-3 gap-2">
            <div>
              <label className="text-[10px] text-slate-600">Ruhepuls morgens</label>
              <input type="number" value={restingHR} onChange={e => setRestingHR(e.target.value)}
                placeholder="bpm"
                className="w-full mt-0.5 px-2 py-2 border border-slate-300 rounded-lg text-sm" />
            </div>
            <div>
              <label className="text-[10px] text-slate-600">Schlaf (h)</label>
              <input type="number" min="0" max="14" step="0.5" value={sleep} onChange={e => setSleep(e.target.value)}
                className="w-full mt-0.5 px-2 py-2 border border-slate-300 rounded-lg text-sm" />
            </div>
            <div>
              <label className="text-[10px] text-slate-600">Energie /10</label>
              <input type="number" min="1" max="10" value={energy} onChange={e => setEnergy(e.target.value)}
                className="w-full mt-0.5 px-2 py-2 border border-slate-300 rounded-lg text-sm" />
            </div>
          </div>

          <div>
            <div className="flex justify-between mb-1.5">
              <label className="text-xs text-slate-600 font-semibold">Müdigkeit/Muskelkater</label>
              <div className="flex items-center gap-2">
                <span className={`px-2 py-0.5 rounded text-xs font-bold text-white ${soreColor(soreness)}`}>{soreness}</span>
                <span className="text-xs text-slate-600">{soreness <= 3 ? 'frisch' : soreness <= 6 ? 'normal' : soreness <= 8 ? 'müde' : 'platt'}</span>
              </div>
            </div>
            <input type="range" min="1" max="10" value={soreness} onChange={e => setSoreness(parseInt(e.target.value))}
              className="w-full accent-amber-600" />
          </div>

          <textarea value={notes} onChange={e => setNotes(e.target.value)} rows="2"
            placeholder="Notizen zum Tag..."
            className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm" />

          <button onClick={submit} className="w-full py-2.5 bg-blue-700 text-white rounded-lg font-semibold">
            Speichern
          </button>
        </div>
      </Card>

      <Card>
        <SectionTitle icon={Calendar}>Letzte Einträge</SectionTitle>
        <div className="space-y-1.5 max-h-72 overflow-y-auto">
          {dailyLogs.slice(0, 10).map((log, i) => {
            const st = log.status || (log.completed === false ? 'missed' : 'planned');
            const stIcon = st === 'more' ? '+' : st === 'less' ? '−' : st === 'missed' ? '✗' : '✓';
            const stColor = st === 'more' ? 'bg-blue-100 text-blue-700' :
                            st === 'less' ? 'bg-amber-100 text-amber-700' :
                            st === 'missed' ? 'bg-red-100 text-red-700' :
                            'bg-emerald-100 text-emerald-700';
            return (
              <div key={i} className="flex justify-between py-1.5 border-b border-slate-100 last:border-0">
                <div className="flex-1">
                  <div className="text-sm font-medium">{formatDateShort(log.date)}</div>
                  <div className="text-xs text-slate-500">
                    {log.planned || '—'}
                    {log.actualDuration && ` · ${log.actualDuration} min tatsächlich`}
                  </div>
                  {log.notes && <div className="text-xs italic text-slate-600 mt-0.5">"{log.notes}"</div>}
                </div>
                <div className="flex flex-col items-end gap-0.5 ml-2 text-xs">
                  <div className="flex gap-1.5 items-center">
                    <span className={`px-1.5 py-0.5 rounded font-bold ${stColor}`}>{stIcon}</span>
                    {log.rpe && <span className="text-slate-500">RPE {log.rpe}</span>}
                  </div>
                  <div className="flex gap-1.5 text-slate-500 text-[10px]">
                    {log.restingHR && <span>❤️{log.restingHR}</span>}
                    <span>⚡{log.energy}</span>
                    <span>💤{log.sleep}h</span>
                  </div>
                </div>
              </div>
            );
          })}
          {dailyLogs.length === 0 && <div className="text-sm text-slate-500 py-4 text-center">Noch keine Einträge.</div>}
        </div>
      </Card>
    </>
  );
};

const WeightSub = ({ weightLog, setWeightLog }) => {
  const [date, setDate] = useState(todayISO());
  const [weight, setWeight] = useState('');
  const [waist, setWaist] = useState('');

  const submit = async () => {
    if (!weight) return;
    const entry = { date, weight: parseFloat(weight), waist: waist ? parseFloat(waist) : null };
    const filtered = weightLog.filter(w => w.date !== date);
    await setWeightLog([...filtered, entry].sort((a, b) => a.date.localeCompare(b.date)));
    setWeight(''); setWaist('');
    alert('Gespeichert!');
  };

  const chartData = weightLog.map(w => ({ date: formatDateShort(w.date), weight: w.weight }));

  return (
    <>
      <Card>
        <SectionTitle icon={Scale}>Neues Gewicht</SectionTitle>
        <div className="grid grid-cols-2 gap-2 mb-2">
          <input type="date" value={date} onChange={e => setDate(e.target.value)}
            className="px-3 py-2 border border-slate-300 rounded-lg text-sm" />
          <input type="number" step="0.1" value={weight} onChange={e => setWeight(e.target.value)}
            placeholder="kg" className="px-3 py-2 border border-slate-300 rounded-lg text-sm" />
        </div>
        <input type="number" step="0.5" value={waist} onChange={e => setWaist(e.target.value)}
          placeholder="Taille (cm) optional" className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm mb-2" />
        <button onClick={submit} className="w-full py-2.5 bg-blue-700 text-white rounded-lg font-semibold">Speichern</button>
      </Card>
      {weightLog.length > 0 && (
        <Card>
          <SectionTitle icon={TrendingUp}>Verlauf</SectionTitle>
          <div style={{ width: '100%', height: 200 }}>
            <ResponsiveContainer>
              <LineChart data={chartData} margin={{ top: 5, right: 10, left: -20, bottom: 5 }}>
                <CartesianGrid strokeDasharray="3 3" stroke="#e2e8f0" />
                <XAxis dataKey="date" fontSize={10} />
                <YAxis fontSize={10} domain={['dataMin - 1', 'dataMax + 1']} />
                <Tooltip />
                <ReferenceLine y={78.2} stroke="#94a3b8" strokeDasharray="3 3" />
                <ReferenceLine y={74} stroke="#10b981" strokeDasharray="3 3" />
                <Line type="monotone" dataKey="weight" stroke="#1d4ed8" strokeWidth={2} dot={{ r: 3 }} />
              </LineChart>
            </ResponsiveContainer>
          </div>
        </Card>
      )}
      <Card>
        <SectionTitle icon={Calendar}>Alle Einträge</SectionTitle>
        <div className="space-y-1 max-h-64 overflow-y-auto">
          {weightLog.slice().reverse().map((w, i) => {
            const diff = w.weight - 78.2;
            return (
              <div key={i} className="flex justify-between py-1.5 border-b border-slate-100 last:border-0">
                <div className="text-sm">{formatDateShort(w.date)}</div>
                <div className="flex gap-2">
                  <span className="font-semibold text-sm">{w.weight.toFixed(1)} kg</span>
                  <span className={`text-xs font-medium ${diff > 0 ? 'text-amber-600' : diff < 0 ? 'text-emerald-600' : 'text-slate-500'}`}>
                    {diff > 0 ? '+' : ''}{diff.toFixed(1)}
                  </span>
                </div>
              </div>
            );
          })}
        </div>
      </Card>
    </>
  );
};

const PROGRAM_A = [
  { name: 'Kniebeuge', target: '4×5-6', sets: 4 },
  { name: 'Bankdrücken', target: '3×5-6', sets: 3 },
  { name: 'Rumänisches Kreuzheben', target: '3×6-8', sets: 3 },
  { name: 'Klimmzug / Latzug', target: '3×6-10', sets: 3 },
  { name: 'Plank', target: '3×45-60s', sets: 3 },
];
const PROGRAM_B = [
  { name: 'Hip Thrust', target: '3×6-8', sets: 3 },
  { name: 'Schulterdrücken', target: '3×6-8', sets: 3 },
  { name: 'Bulgarian Split Squat', target: '3×8/B', sets: 3 },
  { name: 'Langhantel-Rudern', target: '3×6-8', sets: 3 },
  { name: 'Hängendes Beinheben', target: '3×10', sets: 3 },
];

const StrengthSub = ({ strengthLog, setStrengthLog }) => {
  const [program, setProgram] = useState('A');
  const [date, setDate] = useState(todayISO());
  const [setData, setSetData] = useState({});
  const exercises = program === 'A' ? PROGRAM_A : PROGRAM_B;
  const lastSession = strengthLog.find(s => s.program === program);

  const submit = async () => {
    const session = {
      date, program,
      exercises: exercises.map((ex, i) => ({
        name: ex.name,
        sets: Array.from({ length: ex.sets }, (_, j) => ({
          weight: setData[`${i}-${j}-w`] || '',
          reps: setData[`${i}-${j}-r`] || ''
        }))
      }))
    };
    await setStrengthLog([...strengthLog, session].sort((a, b) => b.date.localeCompare(a.date)));
    setSetData({});
    alert('Session gespeichert!');
  };

  return (
    <>
      <Card>
        <SectionTitle icon={Dumbbell}>Krafteinheit loggen</SectionTitle>
        <div className="grid grid-cols-2 gap-2 mb-3">
          <input type="date" value={date} onChange={e => setDate(e.target.value)}
            className="px-3 py-2 border border-slate-300 rounded-lg text-sm" />
          <div className="flex gap-1">
            <button onClick={() => setProgram('A')}
              className={`flex-1 py-2 rounded-lg border font-medium text-sm ${program === 'A' ? 'bg-blue-700 text-white border-blue-700' : 'bg-white text-slate-600 border-slate-300'}`}>A</button>
            <button onClick={() => setProgram('B')}
              className={`flex-1 py-2 rounded-lg border font-medium text-sm ${program === 'B' ? 'bg-blue-700 text-white border-blue-700' : 'bg-white text-slate-600 border-slate-300'}`}>B</button>
          </div>
        </div>
        <div className="space-y-3">
          {exercises.map((ex, i) => {
            const lastEx = lastSession?.exercises.find(e => e.name === ex.name);
            return (
              <div key={i} className="bg-slate-50 rounded-lg p-2.5">
                <div className="flex items-center justify-between mb-1">
                  <div className="font-semibold text-sm">{ex.name}</div>
                  <span className="text-xs text-slate-500">{ex.target}</span>
                </div>
                {lastEx && <div className="text-[10px] text-slate-500 mb-1.5">Letzte: {lastEx.sets.map(s => `${s.weight || '–'}×${s.reps || '–'}`).join(' · ')}</div>}
                <div className="grid grid-cols-2 gap-1.5">
                  {Array.from({ length: ex.sets }, (_, j) => (
                    <div key={j} className="flex gap-1 items-center">
                      <span className="text-[10px] text-slate-500 w-5">S{j+1}</span>
                      <input type="number" placeholder="kg" step="0.5"
                        value={setData[`${i}-${j}-w`] || ''}
                        onChange={e => setSetData({ ...setData, [`${i}-${j}-w`]: e.target.value })}
                        className="w-full px-1.5 py-1 text-xs border border-slate-300 rounded" />
                      <span className="text-[10px] text-slate-400">×</span>
                      <input type="number" placeholder="Wdh"
                        value={setData[`${i}-${j}-r`] || ''}
                        onChange={e => setSetData({ ...setData, [`${i}-${j}-r`]: e.target.value })}
                        className="w-full px-1.5 py-1 text-xs border border-slate-300 rounded" />
                    </div>
                  ))}
                </div>
              </div>
            );
          })}
        </div>
        <button onClick={submit} className="w-full mt-3 py-2.5 bg-blue-700 text-white rounded-lg font-semibold">Speichern</button>
      </Card>
      <Card>
        <SectionTitle icon={Calendar}>Letzte Sessions</SectionTitle>
        <div className="space-y-2 max-h-80 overflow-y-auto">
          {strengthLog.slice(0, 8).map((s, i) => (
            <div key={i} className="border-b border-slate-100 pb-2 last:border-0">
              <div className="flex justify-between text-xs">
                <span className="font-semibold">Programm {s.program}</span>
                <span className="text-slate-500">{formatDateShort(s.date)}</span>
              </div>
              {s.exercises.slice(0, 3).map((ex, j) => (
                <div key={j} className="text-[11px] text-slate-600 mt-0.5">
                  {ex.name}: {ex.sets.filter(set => set.weight || set.reps).map(set => `${set.weight}×${set.reps}`).join(' / ') || '—'}
                </div>
              ))}
            </div>
          ))}
          {strengthLog.length === 0 && <div className="text-sm text-slate-500 py-4 text-center">Noch keine Sessions.</div>}
        </div>
      </Card>
    </>
  );
};

const SWIM_MILESTONES = [
  { goal: '25 m sauber', distance: 25, targetWeek: 2 },
  { goal: '100 m durchgehend', distance: 100, targetWeek: 3 },
  { goal: '200 m durchgehend', distance: 200, targetWeek: 5 },
  { goal: '300 m durchgehend', distance: 300, targetWeek: 6 },
  { goal: '500 m durchgehend', distance: 500, targetWeek: 7 },
  { goal: '750 m (Sprint-Distanz)', distance: 750, targetWeek: 8 },
  { goal: '1000 m durchgehend', distance: 1000, targetWeek: 10 },
  { goal: '1500 m (Renn-Distanz!)', distance: 1500, targetWeek: 11 },
];

const SwimSub = ({ swimLog, setSwimLog }) => {
  const [date, setDate] = useState(todayISO());
  const [duration, setDuration] = useState('');
  const [longest, setLongest] = useState('');
  const [total, setTotal] = useState('');
  const [location, setLocation] = useState('Pool');
  const [notes, setNotes] = useState('');

  const submit = async () => {
    if (!longest) return;
    const entry = { date, duration: duration ? parseInt(duration) : null, longestContinuous: parseInt(longest), totalDistance: total ? parseInt(total) : null, location, notes };
    await setSwimLog([...swimLog, entry].sort((a, b) => b.date.localeCompare(a.date)));
    setDuration(''); setLongest(''); setTotal(''); setNotes('');
    alert('Gespeichert!');
  };

  const longestEver = swimLog.reduce((max, s) => Math.max(max, s.longestContinuous || 0), 0);

  return (
    <>
      <Card>
        <SectionTitle icon={Waves}>Schwimm-Einheit</SectionTitle>
        <div className="grid grid-cols-2 gap-2 mb-2">
          <input type="date" value={date} onChange={e => setDate(e.target.value)} className="px-3 py-2 border border-slate-300 rounded-lg text-sm" />
          <input type="number" value={duration} onChange={e => setDuration(e.target.value)} placeholder="Dauer (min)" className="px-3 py-2 border border-slate-300 rounded-lg text-sm" />
        </div>
        <div className="mb-2">
          <label className="text-xs font-semibold text-slate-700">⭐ Längste Distanz am Stück (m)</label>
          <input type="number" value={longest} onChange={e => setLongest(e.target.value)} placeholder="z.B. 100, 200..." className="w-full mt-0.5 px-3 py-2 border-2 border-blue-300 rounded-lg" />
        </div>
        <div className="grid grid-cols-2 gap-2 mb-2">
          <input type="number" value={total} onChange={e => setTotal(e.target.value)} placeholder="Total (m)" className="px-3 py-2 border border-slate-300 rounded-lg text-sm" />
          <select value={location} onChange={e => setLocation(e.target.value)} className="px-3 py-2 border border-slate-300 rounded-lg text-sm">
            <option>Pool</option>
            <option>Freiwasser</option>
          </select>
        </div>
        <textarea value={notes} onChange={e => setNotes(e.target.value)} rows="2" placeholder="Notizen..." className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm mb-2" />
        <button onClick={submit} className="w-full py-2.5 bg-blue-700 text-white rounded-lg font-semibold">Speichern</button>
      </Card>
      <Card>
        <SectionTitle icon={Target}>Meilensteine</SectionTitle>
        <div className="space-y-1.5">
          {SWIM_MILESTONES.map((m, i) => {
            const reached = longestEver >= m.distance;
            return (
              <div key={i} className={`flex justify-between py-1.5 px-2.5 rounded-lg ${reached ? 'bg-emerald-50' : 'bg-slate-50'}`}>
                <div className="flex items-center gap-2">
                  {reached ? <CheckCircle size={16} className="text-emerald-600" /> : <div className="w-4 h-4 rounded-full border-2 border-slate-300" />}
                  <span className={`text-sm ${reached ? 'text-emerald-800 font-medium' : 'text-slate-700'}`}>{m.goal}</span>
                </div>
                <span className="text-[10px] text-slate-500 self-center">W{m.targetWeek}</span>
              </div>
            );
          })}
        </div>
      </Card>
      <Card>
        <SectionTitle icon={Calendar}>Letzte Einheiten</SectionTitle>
        <div className="space-y-1.5 max-h-64 overflow-y-auto">
          {swimLog.slice(0, 8).map((s, i) => (
            <div key={i} className="border-b border-slate-100 pb-1.5 last:border-0">
              <div className="text-sm font-medium">{formatDateShort(s.date)} · {s.location}</div>
              <div className="text-xs text-slate-600">Längste: <span className="font-bold text-blue-700">{s.longestContinuous} m</span>{s.totalDistance && ` · Total: ${s.totalDistance} m`}{s.duration && ` · ${s.duration} min`}</div>
              {s.notes && <div className="text-xs italic text-slate-500">"{s.notes}"</div>}
            </div>
          ))}
        </div>
      </Card>
    </>
  );
};

// CARDIO TESTS
const CARDIO_TESTS = [
  { id: '5k-baseline', name: '5K Baseline', type: 'run', plannedDate: '2026-05-18', week: 1 },
  { id: '5k-phase1', name: '5K Phase 1 Ende', type: 'run', plannedDate: '2026-06-08', week: 4 },
  { id: '5k-phase2', name: '5K Phase 2 Ende', type: 'run', plannedDate: '2026-07-06', week: 8 },
  { id: '5k-phase3', name: '5K Phase 3 Mitte', type: 'run', plannedDate: '2026-07-26', week: 11 },
  { id: '5k-pre-race', name: '5K vor Rennen', type: 'run', plannedDate: '2026-08-16', week: 14 },
  { id: 'bike-baseline', name: 'Rad 60 min Baseline', type: 'bike', plannedDate: '2026-05-17', week: 1 },
  { id: 'bike-phase2', name: 'Rad 75 min', type: 'bike', plannedDate: '2026-06-28', week: 7 },
  { id: 'bike-phase3', name: 'Rad 90+ min', type: 'bike', plannedDate: '2026-07-26', week: 11 },
  { id: 'race-sim', name: 'Race Simulation 35 km', type: 'bike', plannedDate: '2026-08-08', week: 13 },
  { id: 'brick-1', name: 'Erster Brick', type: 'brick', plannedDate: '2026-06-21', week: 6 },
  { id: 'brick-2', name: 'Phase 3 Brick', type: 'brick', plannedDate: '2026-07-12', week: 9 },
  { id: 'mini-tri', name: 'Mini-Tri Sim', type: 'brick', plannedDate: '2026-07-25', week: 11 },
];

const CardioSub = ({ cardioTests, setCardioTests }) => {
  const [selected, setSelected] = useState(null);
  const [value, setValue] = useState('');
  const [notes, setNotes] = useState('');

  const submit = async () => {
    if (!selected || !value) return;
    const test = CARDIO_TESTS.find(t => t.id === selected);
    const entry = { id: selected, name: test.name, type: test.type, value, notes, date: todayISO() };
    const filtered = cardioTests.filter(t => t.id !== selected);
    await setCardioTests([...filtered, entry]);
    setSelected(null); setValue(''); setNotes('');
    alert('Test gespeichert!');
  };

  const grouped = { run: CARDIO_TESTS.filter(t => t.type === 'run'), bike: CARDIO_TESTS.filter(t => t.type === 'bike'), brick: CARDIO_TESTS.filter(t => t.type === 'brick') };

  return (
    <>
      <Card>
        <SectionTitle icon={Timer}>Cardio-Test eintragen</SectionTitle>
        <select value={selected || ''} onChange={e => setSelected(e.target.value)} className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm mb-2">
          <option value="">Test auswählen...</option>
          {CARDIO_TESTS.map(t => <option key={t.id} value={t.id}>{t.name} ({t.plannedDate})</option>)}
        </select>
        <input type="text" value={value} onChange={e => setValue(e.target.value)} placeholder="Ergebnis (z.B. '25:30', '32 km/h', '1:35h')" className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm mb-2" />
        <textarea value={notes} onChange={e => setNotes(e.target.value)} rows="2" placeholder="Wetter, Bedingungen, Gefühl..." className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm mb-2" />
        <button onClick={submit} className="w-full py-2.5 bg-blue-700 text-white rounded-lg font-semibold">Speichern</button>
      </Card>
      {['run', 'bike', 'brick'].map(type => (
        <Card key={type}>
          <SectionTitle icon={type === 'run' ? Activity : type === 'bike' ? TrendingUp : Target}>
            {type === 'run' ? '5K Lauf-Tests' : type === 'bike' ? 'Rad-Benchmarks' : 'Brick-Tests'}
          </SectionTitle>
          <div className="space-y-1.5">
            {grouped[type].map(t => {
              const done = cardioTests.find(c => c.id === t.id);
              return (
                <div key={t.id} className={`p-2 rounded-lg ${done ? 'bg-emerald-50' : 'bg-slate-50'}`}>
                  <div className="flex justify-between text-sm">
                    <span className={done ? 'font-semibold text-emerald-800' : 'text-slate-700'}>{t.name}</span>
                    <span className="text-xs text-slate-500">{t.plannedDate.slice(5)} · W{t.week}</span>
                  </div>
                  {done && <div className="text-xs text-emerald-700 mt-0.5">{done.value} {done.notes && `· ${done.notes}`}</div>}
                </div>
              );
            })}
          </div>
        </Card>
      ))}
    </>
  );
};

// NUTRITION
const NutritionSub = ({ nutritionLogs, setNutritionLogs }) => {
  const [date, setDate] = useState(todayISO());
  const todayPlan = PLAN_BY_DATE[date];
  const phase = getCurrentPhase(new Date(date));
  const dayType = todayPlan?.type === 'race' ? 'hart' : (todayPlan?.type || 'rest');
  const macros = PHASE_MACROS[phase.num || 1]?.[dayType];

  const existing = nutritionLogs.find(n => n.date === date) || { date, kcal: '', protein: '', water: '', hitProtein: false, hitKcal: false };
  const [entry, setEntry] = useState(existing);

  useEffect(() => {
    const e = nutritionLogs.find(n => n.date === date) || { date, kcal: '', protein: '', water: '', hitProtein: false, hitKcal: false };
    setEntry(e);
  }, [date, nutritionLogs]);

  const submit = async () => {
    const filtered = nutritionLogs.filter(n => n.date !== date);
    await setNutritionLogs([...filtered, entry].sort((a, b) => b.date.localeCompare(a.date)));
    alert('Gespeichert!');
  };

  return (
    <>
      <Card>
        <SectionTitle icon={Apple}>Ernährungs-Adherence</SectionTitle>
        <input type="date" value={date} onChange={e => setDate(e.target.value)}
          className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm mb-3" />
        {macros && (
          <div className="bg-blue-50 rounded-lg p-2.5 mb-3">
            <div className="flex items-center gap-2 mb-1">
              <TypeBadge type={dayType} />
              <span className="text-xs text-slate-600">Ziel-Makros</span>
            </div>
            <div className="grid grid-cols-4 gap-1.5 text-center mt-1">
              <div><div className="text-[10px] text-slate-500">kcal</div><div className="font-bold text-sm">{macros.kcal}</div></div>
              <div><div className="text-[10px] text-slate-500">P</div><div className="font-bold text-sm">{macros.p}g</div></div>
              <div><div className="text-[10px] text-slate-500">F</div><div className="font-bold text-sm">{macros.f}g</div></div>
              <div><div className="text-[10px] text-slate-500">C</div><div className="font-bold text-sm">{macros.c}g</div></div>
            </div>
          </div>
        )}
        <div className="space-y-2">
          <div className="grid grid-cols-2 gap-2">
            <div>
              <label className="text-xs text-slate-600">Tatsächliche kcal</label>
              <input type="number" value={entry.kcal} onChange={e => setEntry({ ...entry, kcal: e.target.value })} className="w-full mt-0.5 px-3 py-2 border border-slate-300 rounded-lg text-sm" />
            </div>
            <div>
              <label className="text-xs text-slate-600">Protein (g)</label>
              <input type="number" value={entry.protein} onChange={e => setEntry({ ...entry, protein: e.target.value })} className="w-full mt-0.5 px-3 py-2 border border-slate-300 rounded-lg text-sm" />
            </div>
          </div>
          <div>
            <label className="text-xs text-slate-600">Wasser (Liter)</label>
            <input type="number" step="0.25" value={entry.water} onChange={e => setEntry({ ...entry, water: e.target.value })} className="w-full mt-0.5 px-3 py-2 border border-slate-300 rounded-lg text-sm" />
          </div>
          <div className="flex gap-2 pt-2">
            <button onClick={() => setEntry({ ...entry, hitProtein: !entry.hitProtein })}
              className={`flex-1 py-2 rounded-lg border text-xs font-medium ${entry.hitProtein ? 'bg-emerald-100 border-emerald-300 text-emerald-800' : 'bg-white border-slate-300 text-slate-600'}`}>
              {entry.hitProtein ? '✓' : '○'} Protein-Ziel
            </button>
            <button onClick={() => setEntry({ ...entry, hitKcal: !entry.hitKcal })}
              className={`flex-1 py-2 rounded-lg border text-xs font-medium ${entry.hitKcal ? 'bg-emerald-100 border-emerald-300 text-emerald-800' : 'bg-white border-slate-300 text-slate-600'}`}>
              {entry.hitKcal ? '✓' : '○'} kcal im Bereich
            </button>
          </div>
          <button onClick={submit} className="w-full py-2.5 bg-blue-700 text-white rounded-lg font-semibold">Speichern</button>
        </div>
      </Card>
      <Card>
        <SectionTitle icon={TrendingUp}>Phase-Übersicht Makros</SectionTitle>
        <div className="overflow-x-auto">
          <table className="w-full text-xs">
            <thead className="bg-slate-50">
              <tr><th className="p-1.5 text-left">Phase</th><th className="p-1.5">Tag</th><th className="p-1.5">kcal</th><th className="p-1.5">P</th><th className="p-1.5">F</th><th className="p-1.5">C</th></tr>
            </thead>
            <tbody>
              {[1, 2, 3].map(p => (['hart', 'moderat', 'rest']).map(t => {
                const m = PHASE_MACROS[p][t];
                return <tr key={`${p}-${t}`} className="border-t border-slate-100"><td className="p-1.5 font-medium">P{p}</td><td className="p-1.5 capitalize">{t}</td><td className="p-1.5 text-center font-semibold">{m.kcal}</td><td className="p-1.5 text-center">{m.p}</td><td className="p-1.5 text-center">{m.f}</td><td className="p-1.5 text-center">{m.c}</td></tr>;
              }))}
            </tbody>
          </table>
        </div>
      </Card>
      <Card>
        <SectionTitle icon={Calendar}>Letzte Tage Adherence</SectionTitle>
        <div className="space-y-1 max-h-48 overflow-y-auto">
          {nutritionLogs.slice(0, 10).map((n, i) => (
            <div key={i} className="flex justify-between py-1.5 border-b border-slate-100 last:border-0 text-xs">
              <div>{formatDateShort(n.date)}</div>
              <div className="flex gap-1.5">
                <span className={`px-1.5 py-0.5 rounded ${n.hitProtein ? 'bg-emerald-100 text-emerald-700' : 'bg-slate-100 text-slate-500'}`}>P</span>
                <span className={`px-1.5 py-0.5 rounded ${n.hitKcal ? 'bg-emerald-100 text-emerald-700' : 'bg-slate-100 text-slate-500'}`}>kcal</span>
              </div>
            </div>
          ))}
        </div>
      </Card>
    </>
  );
};

// ============ MOBILITY ============
const MobilitySub = ({ mobilityLogs, setMobilityLogs }) => {
  const [date, setDate] = useState(todayISO());
  const [duration, setDuration] = useState(15);
  const [type, setType] = useState('Stretching');
  const [notes, setNotes] = useState('');

  useEffect(() => {
    const existing = mobilityLogs.find(l => l.date === date);
    if (existing) {
      setDuration(existing.duration);
      setType(existing.type || 'Stretching');
      setNotes(existing.notes || '');
    } else {
      setDuration(15); setType('Stretching'); setNotes('');
    }
  }, [date]);

  const submit = async () => {
    const entry = { date, duration: parseInt(duration), type, notes };
    const filtered = mobilityLogs.filter(l => l.date !== date);
    await setMobilityLogs([...filtered, entry].sort((a, b) => b.date.localeCompare(a.date)));
    alert('Gespeichert!');
  };

  // Streak calculation
  const streak = useMemo(() => {
    let count = 0;
    const checkDate = new Date();
    const todayEntry = mobilityLogs.find(l => l.date === todayISO() && l.duration > 0);
    if (!todayEntry) checkDate.setDate(checkDate.getDate() - 1);
    for (let i = 0; i < 365; i++) {
      const iso = checkDate.toISOString().slice(0, 10);
      const entry = mobilityLogs.find(l => l.date === iso && l.duration > 0);
      if (entry) {
        count++;
        checkDate.setDate(checkDate.getDate() - 1);
      } else break;
    }
    return count;
  }, [mobilityLogs]);

  // This week (Mon-Sun)
  const weekDays = useMemo(() => {
    const today = new Date();
    const dow = today.getDay(); // 0 Sun, 1 Mon, ...
    const mondayOffset = dow === 0 ? -6 : 1 - dow;
    const monday = new Date(today);
    monday.setDate(today.getDate() + mondayOffset);
    const days = [];
    for (let i = 0; i < 7; i++) {
      const d = new Date(monday);
      d.setDate(d.getDate() + i);
      const iso = d.toISOString().slice(0, 10);
      const entry = mobilityLogs.find(l => l.date === iso);
      days.push({
        day: ['Mo','Di','Mi','Do','Fr','Sa','So'][i],
        iso,
        duration: entry?.duration || 0,
        future: d > today,
        isToday: iso === todayISO(),
      });
    }
    return days;
  }, [mobilityLogs]);

  const weekTotal = weekDays.reduce((s, d) => s + d.duration, 0);
  const totalAllTime = mobilityLogs.reduce((s, m) => s + (m.duration || 0), 0);

  return (
    <>
      <Card>
        <SectionTitle icon={Activity}>Mobility-Einheit</SectionTitle>
        <input type="date" value={date} onChange={e => setDate(e.target.value)}
          className="w-full px-3 py-2 border border-slate-300 rounded-lg mb-3" />

        <div className="mb-3">
          <label className="text-xs text-slate-600 mb-1.5 block font-semibold">Dauer (min)</label>
          <div className="grid grid-cols-5 gap-1.5 mb-2">
            {[0, 10, 15, 20, 30].map(min => (
              <button key={min} onClick={() => setDuration(min)}
                className={`py-2 rounded-lg border text-sm font-medium ${parseInt(duration) === min ? 'bg-blue-700 text-white border-blue-700' : 'bg-white text-slate-600 border-slate-300'}`}>
                {min}
              </button>
            ))}
          </div>
          <input type="number" value={duration} onChange={e => setDuration(e.target.value)}
            placeholder="oder eigene Minutenzahl"
            className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm" />
        </div>

        <div className="mb-3">
          <label className="text-xs text-slate-600 mb-1.5 block font-semibold">Art</label>
          <div className="grid grid-cols-4 gap-1.5">
            {['Stretching', 'Yoga', 'Foam Roll', 'Sonstiges'].map(t => (
              <button key={t} onClick={() => setType(t)}
                className={`py-1.5 rounded-lg border text-[11px] font-medium ${type === t ? 'bg-blue-700 text-white border-blue-700' : 'bg-white text-slate-600 border-slate-300'}`}>
                {t}
              </button>
            ))}
          </div>
        </div>

        <textarea value={notes} onChange={e => setNotes(e.target.value)} rows="2"
          placeholder="Fokus auf was? (Hüfte, Schultern, Waden...)"
          className="w-full px-3 py-2 border border-slate-300 rounded-lg text-sm mb-3" />

        <button onClick={submit} className="w-full py-2.5 bg-blue-700 text-white rounded-lg font-semibold">
          Speichern
        </button>
      </Card>

      <Card>
        <SectionTitle icon={TrendingUp}>Übersicht</SectionTitle>
        <div className="grid grid-cols-3 gap-2 mb-3">
          <div className="bg-orange-50 rounded-lg p-2.5 text-center">
            <div className="text-2xl">🔥</div>
            <div className="font-bold text-lg text-orange-700">{streak}</div>
            <div className="text-[10px] text-slate-600">Streak (Tage)</div>
          </div>
          <div className="bg-blue-50 rounded-lg p-2.5 text-center">
            <div className="font-bold text-lg text-blue-700">{weekTotal}</div>
            <div className="text-[10px] text-slate-600">min · diese Woche</div>
            <div className="text-[9px] text-slate-500">Ziel: 60+</div>
          </div>
          <div className="bg-emerald-50 rounded-lg p-2.5 text-center">
            <div className="font-bold text-lg text-emerald-700">{totalAllTime}</div>
            <div className="text-[10px] text-slate-600">min · gesamt</div>
          </div>
        </div>
        <div className="flex gap-1 justify-between">
          {weekDays.map((d, i) => (
            <div key={i} className="flex-1 text-center">
              <div className="text-[10px] text-slate-500 mb-0.5">{d.day}</div>
              <div className={`rounded-lg py-1.5 text-xs font-bold ${
                d.future ? 'bg-slate-50 text-slate-300' :
                d.duration > 0 ? 'bg-emerald-100 text-emerald-700' :
                d.isToday ? 'bg-amber-100 text-amber-700' :
                'bg-slate-100 text-slate-400'
              }`}>
                {d.future ? '·' : d.duration > 0 ? `${d.duration}'` : '–'}
              </div>
            </div>
          ))}
        </div>
      </Card>

      <Card className="bg-blue-50 border-blue-200">
        <SectionTitle icon={BookOpen}>💡 Mobility-Empfehlungen</SectionTitle>
        <ul className="text-xs space-y-1.5 text-slate-700">
          <li>• <strong>10–15 min nach Krafttraining:</strong> Hüfte und Schultern — wichtig fürs Schwimmen</li>
          <li>• <strong>10 min nach harten Läufen:</strong> Waden, Hamstrings, Hüftbeuger — verhindert Schienbein-/Kniebeschwerden</li>
          <li>• <strong>20 min Yoga 1×/Woche:</strong> ideal für Recovery-Tag (Freitag), volle Ganzkörper-Mobility</li>
          <li>• <strong>Foam Roll täglich 5 min:</strong> Quads, IT-Band, oberer Rücken — kostenlose Recovery</li>
          <li>• <strong>Schulter-Mobility kritisch fürs Kraulen:</strong> Wandkreise + Türrahmen-Dehnung täglich 3 min</li>
          <li>• <strong>Pro-Tipp:</strong> Mobility direkt nach dem Training, solange Muskeln warm — nicht später am Abend</li>
        </ul>
      </Card>

      <Card>
        <SectionTitle icon={Calendar}>Letzte Einheiten</SectionTitle>
        <div className="space-y-1.5 max-h-64 overflow-y-auto">
          {mobilityLogs.slice(0, 12).map((m, i) => (
            <div key={i} className="flex justify-between py-1.5 border-b border-slate-100 last:border-0">
              <div className="flex-1">
                <div className="text-sm font-medium">{formatDateShort(m.date)} · {m.type}</div>
                {m.notes && <div className="text-xs italic text-slate-500 mt-0.5">"{m.notes}"</div>}
              </div>
              <div className="text-sm font-semibold text-blue-700 ml-2">{m.duration} min</div>
            </div>
          ))}
          {mobilityLogs.length === 0 && <div className="text-sm text-slate-500 py-4 text-center">Noch keine Mobility-Einheiten.</div>}
        </div>
      </Card>
    </>
  );
};

// ============ WISSEN TAB ============
const Accordion = ({ title, icon: Icon, children, defaultOpen = false }) => {
  const [open, setOpen] = useState(defaultOpen);
  return (
    <Card>
      <button onClick={() => setOpen(!open)} className="w-full flex items-center justify-between text-left">
        <div className="flex items-center gap-2">
          {Icon && <Icon size={18} className="text-blue-700" />}
          <h3 className="font-bold text-slate-800">{title}</h3>
        </div>
        {open ? <ChevronDown size={18} className="text-slate-400" /> : <ChevronRight size={18} className="text-slate-400" />}
      </button>
      {open && <div className="mt-3 pt-3 border-t border-slate-100 text-sm text-slate-700 space-y-2">{children}</div>}
    </Card>
  );
};

const WissenTab = () => {
  return (
    <div className="space-y-3">
      <Accordion title="Trainingszonen Z1–Z5" icon={Activity} defaultOpen>
        <p className="text-xs italic text-slate-600">80 % deines Volumens in Z1/Z2, 20 % in Z3/Z4. Niemals Z5 außer Wettkampf.</p>
        <div className="overflow-x-auto">
          <table className="w-full text-xs">
            <thead className="bg-slate-50"><tr><th className="p-1.5">Zone</th><th className="p-1.5 text-left">Gefühl</th><th className="p-1.5 text-left">Atmung</th></tr></thead>
            <tbody>
              <tr className="border-t"><td className="p-1.5 font-bold text-center">Z1</td><td className="p-1.5">Sehr locker</td><td className="p-1.5">Wie unterhalten</td></tr>
              <tr className="border-t bg-slate-50"><td className="p-1.5 font-bold text-center">Z2</td><td className="p-1.5">Locker, „könnte ewig"</td><td className="p-1.5">Ganze Sätze</td></tr>
              <tr className="border-t"><td className="p-1.5 font-bold text-center">Z3</td><td className="p-1.5">Komfortabel-hart</td><td className="p-1.5">5-7 Wörter</td></tr>
              <tr className="border-t bg-slate-50"><td className="p-1.5 font-bold text-center">Z4</td><td className="p-1.5">Hart, „Schwelle"</td><td className="p-1.5">Einzelne Wörter</td></tr>
              <tr className="border-t"><td className="p-1.5 font-bold text-center">Z5</td><td className="p-1.5">Sehr hart, max 3-5 min</td><td className="p-1.5">Sprechen unmöglich</td></tr>
            </tbody>
          </table>
        </div>
      </Accordion>

      <Accordion title="Kraftprogramm A – Bein + Push (funktionell)" icon={Dumbbell}>
        <p className="text-xs italic text-slate-600">Fokus: Kraft erhalten im Defizit, Compound-Lifts, keine Isolation. ~40 min inkl. Aufwärmen.</p>
        <ul className="space-y-1.5">
          <li><strong>1. Kniebeuge:</strong> 4×5-6 (schwer). Tiefe bis Parallel, 2-3 min Pause. Hauptlift.</li>
          <li><strong>2. Bankdrücken:</strong> 3×5-6. Kontrolliert ablassen, volle Bewegung.</li>
          <li><strong>3. Rumänisches Kreuzheben:</strong> 3×6-8. Hüfte nach hinten, Rücken gerade.</li>
          <li><strong>4. Klimmzug/Latzug:</strong> 3×6-10. Schulterblätter aktiv ziehen.</li>
          <li><strong>5. Plank:</strong> 3×45-60s. Gerader Körper, Bauchspannung.</li>
        </ul>
      </Accordion>

      <Accordion title="Kraftprogramm B – Hüfte + Pull (funktionell)" icon={Dumbbell}>
        <p className="text-xs italic text-slate-600">Fokus auf athletische Bewegungsmuster, einseitige Stabilität. ~40 min.</p>
        <ul className="space-y-1.5">
          <li><strong>1. Hip Thrust:</strong> 3×6-8 schwer. Schulterblätter auf Bank, Hantel über Hüfte (mit Polster!), Füße flach. Glutes voll anspannen am Top, 1 s halten. 2-3 min Pause. Sicher, max. Glute-Aktivierung → direkt übertragbar auf Lauf-/Rad-Power.</li>
          <li><strong>2. Schulterdrücken:</strong> 3×6-8. Sitzend oder stehend.</li>
          <li><strong>3. Bulgarian Split Squat:</strong> 3×8/Bein. Hinteres Bein erhöht. Triathlon-spezifisch.</li>
          <li><strong>4. Langhantel-Rudern:</strong> 3×6-8. Wichtig für Schwimm-Posture.</li>
          <li><strong>5. Hängendes Beinheben:</strong> 3×10. Core-Pflicht für Triathlon.</li>
        </ul>
        <p className="text-xs text-slate-600 mt-2"><strong>Warum diese Reduktion:</strong> Im Fettverlust-Modus ist „Pump-Volumen" kontraproduktiv (kostet Recovery, hilft nicht). Compound-Lifts halten deine Kraft und Muskulatur, ohne dein Cardio-Training zu ruinieren.</p>
      </Accordion>

      <Accordion title="Schwimm-Drills" icon={Waves}>
        <div>
          <p className="font-semibold text-slate-800">Wasserlage</p>
          <ul className="space-y-1 mt-1">
            <li>• <strong>Sweet-Spot:</strong> Seitenlage, unterer Arm gestreckt, Kopf seitlich auf dem Arm. Locker kicken.</li>
            <li>• <strong>Kickboard nach hinten:</strong> Brett zwischen Knien, Arme vorne. Atme alle 4 Kicks zur Seite.</li>
          </ul>
        </div>
        <div>
          <p className="font-semibold text-slate-800">Atmung</p>
          <ul className="space-y-1 mt-1">
            <li>• <strong>Ausatmen unter Wasser:</strong> Wichtigste Übung. NICHT Luft anhalten! Konstant ausatmen, nur einatmen beim Drehen.</li>
            <li>• <strong>3er-Atmung:</strong> Alle 3 Züge atmen → wechselt automatisch die Seite. Im Wettkampf Pflicht!</li>
          </ul>
        </div>
        <div>
          <p className="font-semibold text-slate-800">Armzug</p>
          <ul className="space-y-1 mt-1">
            <li>• <strong>Catch-up:</strong> Ein Arm bleibt vorne bis der andere ihn einholt. Erzwingt langes Gleiten.</li>
            <li>• <strong>Fingerschleif:</strong> Beim Vorschwung Finger über Wasseroberfläche schleifen. Lehrt hohen Ellbogen.</li>
          </ul>
        </div>
        <div>
          <p className="font-semibold text-slate-800">Freiwasser (ab Woche 9)</p>
          <ul className="space-y-1 mt-1">
            <li>• <strong>Sighting:</strong> Alle 6-8 Züge Kopf vorne raus, Fixpunkt am Ufer/Boje. Üben üben üben.</li>
            <li>• <strong>Neopren:</strong> 20 min vor Start anziehen, mit Vaseline am Hals gegen Scheuern.</li>
            <li>• <strong>Kalter Schock:</strong> Erste 200 m ruhig, locker atmen, nicht panisch werden.</li>
          </ul>
        </div>
      </Accordion>

      <Accordion title="Renntag-Protokoll" icon={Target}>
        <div>
          <p className="font-semibold">Tage vorher:</p>
          <ul className="mt-1 space-y-1">
            <li>• Do/Fr: Carb-Load (8-10 g/kg = 600-800 g/Tag). Pasta, Reis, Brot, Bananen.</li>
            <li>• Fr: Material checken — Reifen, Bremsen, Helm, Brille, Neopren, Chip.</li>
            <li>• Sa: Strecke besichtigen, Race-Briefing, früh schlafen.</li>
          </ul>
        </div>
        <div>
          <p className="font-semibold">Renntag-Frühstück (3 h vorher):</p>
          <p className="mt-1">Haferflocken + Banane + Honig + Toast mit Marmelade + Espresso (~700 kcal). NICHTS Neues testen!</p>
        </div>
        <div>
          <p className="font-semibold">Pacing:</p>
          <ul className="mt-1 space-y-1">
            <li>• <strong>Schwimmen:</strong> Erste 200 m kontrolliert. Z3, nicht Z4. Sighting alle 6-8 Züge.</li>
            <li>• <strong>T1:</strong> Neopren aus, Helm AUF (vor dem Rad anfassen!), Schuhe, los.</li>
            <li>• <strong>Rad:</strong> Z3 konstant, 90-95 % Schwellen-Watt. Trinken alle 15 min, Gel bei km 20.</li>
            <li>• <strong>T2:</strong> Rad weg, Helm AB, Laufschuhe. Erste 1 km langsamer.</li>
            <li>• <strong>Laufen:</strong> Erste 2 km kontrolliert (Z3). km 3-7 Renn-Pace. Letzte 3 km alles geben.</li>
          </ul>
        </div>
      </Accordion>

      <Accordion title="Was tun, wenn..." icon={AlertTriangle}>
        <div>
          <p className="font-semibold">...ich eine Einheit verpasse:</p>
          <p>Eine = ignorieren, weitermachen. Zwei = Woche normal weiter. Drei+ = Plan eine Woche zurücksetzen. NIE nachholen.</p>
        </div>
        <div>
          <p className="font-semibold">...ich krank werde:</p>
          <p>Symptome OBERHALB Hals (Schnupfen): kann leicht trainiert werden. UNTERHALB (Fieber, Husten, Atemnot): KOMPLETTE Pause bis 2 Tage nach letzten Symptom. Training bei Fieber kann Herzmuskelentzündung verursachen.</p>
        </div>
        <div>
          <p className="font-semibold">...ich Schmerzen habe:</p>
          <p>Muskelkater = OK. Stechender Schmerz (Knie, Schienbein, Achilles) = Pause 2-3 Tage. Bleibt: Physio. NICHT „durchlaufen".</p>
        </div>
        <div>
          <p className="font-semibold">...ich am Wochenende kaputt bin:</p>
          <p>Nach harter Woche normal. Nach Deload = Warnsignal. Schlaf? Kalorien? Eisen (Bluttest)? Lieber Ruhetag dazu nehmen.</p>
        </div>
        <div>
          <p className="font-semibold">...ich die Schwimmstrecke nicht schaffe:</p>
          <p>Ende Woche 8: ehrlich. 750 m am Stück → Olympic machbar. Unter 500 m → downgraden auf Sprint. KEIN Gesichtsverlust. Gefinishter Sprint &gt; abgebrochener Olympic.</p>
        </div>
      </Accordion>

      <Accordion title="Tag-Klassifikation für Ernährung" icon={Apple}>
        <p><strong>RUHE-Tag:</strong> Kein strukturiertes Training.</p>
        <p><strong>HARTER Tag — mind. eines davon:</strong></p>
        <ul className="space-y-0.5 ml-2">
          <li>• Endurance-Einheit länger als 60 min</li>
          <li>• Zwei oder mehr Trainingseinheiten</li>
          <li>• Mit Intensität (Intervalle, Schwelle, Tempo, Renn-Pace)</li>
          <li>• Brick-Workout</li>
        </ul>
        <p><strong>MODERATER Tag:</strong> Alles dazwischen — eine einzelne Einheit unter 60 min, oder Kraft alleine.</p>
        <p className="italic text-xs">Im Zweifel moderat. Wochenmittel zählt.</p>
      </Accordion>
    </div>
  );
};

// ============ MEHR TAB ============
const PRsCard = ({ swimLog, strengthLog, manualPRs, setManualPRs }) => {
  // Auto-detected: longest swim
  const longestSwim = swimLog.reduce((max, s) => {
    const dist = s.longestContinuous || 0;
    return dist > max.dist ? { dist, date: s.date } : max;
  }, { dist: 0, date: null });

  // Auto-calculated 1RM via Epley formula
  const calc1RM = (matchName) => {
    let best = { value: 0, date: null };
    strengthLog.forEach(s => {
      s.exercises.forEach(ex => {
        if (ex.name.toLowerCase().includes(matchName.toLowerCase())) {
          ex.sets.forEach(set => {
            const w = parseFloat(set.weight);
            const r = parseInt(set.reps);
            if (w > 0 && r > 0 && r <= 12) {
              const oneRM = w * (1 + r / 30);
              if (oneRM > best.value) best = { value: Math.round(oneRM * 2) / 2, date: s.date };
            }
          });
        }
      });
    });
    return best.value > 0 ? best : null;
  };

  const oneRMs = {
    'Kniebeuge': calc1RM('Kniebeuge'),
    'Hip Thrust': calc1RM('Hip Thrust'),
    'Bankdrücken': calc1RM('Bankdrücken'),
    'Schulterdrücken': calc1RM('Schulterdrücken'),
  };

  const [editingKey, setEditingKey] = useState(null);
  const [tempValue, setTempValue] = useState('');

  const startEdit = (key) => {
    setEditingKey(key);
    setTempValue(manualPRs[key]?.value || '');
  };
  const saveEdit = async (key) => {
    const updated = { ...manualPRs, [key]: tempValue ? { value: tempValue, date: todayISO() } : null };
    await setManualPRs(updated);
    setEditingKey(null);
    setTempValue('');
  };

  const PRRow = ({ label, value, date, manualKey, placeholder }) => (
    <div className="flex justify-between items-center py-1.5 border-b border-slate-100 last:border-0">
      <span className="text-sm text-slate-700 flex-1">{label}</span>
      {editingKey === manualKey ? (
        <div className="flex gap-1 items-center">
          <input type="text" value={tempValue} onChange={e => setTempValue(e.target.value)}
            placeholder={placeholder} autoFocus
            className="w-24 px-2 py-1 border border-slate-300 rounded text-xs" />
          <button onClick={() => saveEdit(manualKey)} className="text-blue-700 font-bold px-1">✓</button>
          <button onClick={() => { setEditingKey(null); setTempValue(''); }} className="text-slate-400 px-1">✗</button>
        </div>
      ) : (
        <div className="flex items-center gap-2">
          {value ? (
            <>
              <span className="font-bold text-sm text-blue-700">{value}</span>
              {date && <span className="text-[10px] text-slate-500">({formatDateShort(date)})</span>}
            </>
          ) : <span className="text-xs text-slate-400">—</span>}
          {manualKey && (
            <button onClick={() => startEdit(manualKey)} className="text-slate-400 text-xs hover:text-blue-600">
              ✏️
            </button>
          )}
        </div>
      )}
    </div>
  );

  return (
    <Card>
      <SectionTitle icon={Target}>Bestzeiten / PRs</SectionTitle>

      <div className="mb-3">
        <div className="text-xs font-bold text-blue-700 mb-1.5">🏊 Schwimmen</div>
        <PRRow label="Längste Distanz am Stück (auto)"
          value={longestSwim.dist > 0 ? `${longestSwim.dist} m` : null}
          date={longestSwim.date} />
        <PRRow label="100 m Zeit" value={manualPRs.swim_100m?.value} date={manualPRs.swim_100m?.date} manualKey="swim_100m" placeholder="1:45" />
        <PRRow label="500 m Zeit" value={manualPRs.swim_500m?.value} date={manualPRs.swim_500m?.date} manualKey="swim_500m" placeholder="9:30" />
        <PRRow label="1000 m Zeit" value={manualPRs.swim_1000m?.value} date={manualPRs.swim_1000m?.date} manualKey="swim_1000m" placeholder="20:00" />
        <PRRow label="1500 m Zeit" value={manualPRs.swim_1500m?.value} date={manualPRs.swim_1500m?.date} manualKey="swim_1500m" placeholder="32:00" />
      </div>

      <div className="mb-3">
        <div className="text-xs font-bold text-blue-700 mb-1.5">🏃 Laufen</div>
        <PRRow label="5 km Zeit" value={manualPRs.run_5k?.value} date={manualPRs.run_5k?.date} manualKey="run_5k" placeholder="25:30" />
        <PRRow label="10 km Zeit" value={manualPRs.run_10k?.value} date={manualPRs.run_10k?.date} manualKey="run_10k" placeholder="52:00" />
        <PRRow label="Längster Lauf (km)" value={manualPRs.run_longest?.value} date={manualPRs.run_longest?.date} manualKey="run_longest" placeholder="12" />
        <PRRow label="Beste Pace (min/km)" value={manualPRs.run_pace?.value} date={manualPRs.run_pace?.date} manualKey="run_pace" placeholder="4:30" />
      </div>

      <div className="mb-3">
        <div className="text-xs font-bold text-blue-700 mb-1.5">🚴 Rad</div>
        <PRRow label="40 km Zeit" value={manualPRs.bike_40k?.value} date={manualPRs.bike_40k?.date} manualKey="bike_40k" placeholder="1:20" />
        <PRRow label="Längste Ausfahrt (km)" value={manualPRs.bike_longest?.value} date={manualPRs.bike_longest?.date} manualKey="bike_longest" placeholder="80" />
        <PRRow label="Beste Ø Speed (km/h)" value={manualPRs.bike_avgSpeed?.value} date={manualPRs.bike_avgSpeed?.date} manualKey="bike_avgSpeed" placeholder="32" />
      </div>

      <div className="mb-2">
        <div className="text-xs font-bold text-blue-700 mb-1.5">💪 Kraft · geschätzte 1RM (auto aus Krafttraining)</div>
        {Object.entries(oneRMs).map(([name, pr]) => (
          <PRRow key={name} label={name} value={pr ? `${pr.value} kg` : null} date={pr?.date} />
        ))}
      </div>

      <div className="mb-2">
        <div className="text-xs font-bold text-blue-700 mb-1.5">🏆 Triathlon-Meilensteine</div>
        <PRRow label="Erste Brick-Einheit" value={manualPRs.first_brick?.value} date={manualPRs.first_brick?.date} manualKey="first_brick" placeholder="z.B. ✓ oder Datum" />
        <PRRow label="Erstes Freiwasser" value={manualPRs.first_owsw?.value} date={manualPRs.first_owsw?.date} manualKey="first_owsw" placeholder="✓" />
        <PRRow label="Erste komplette Olympic" value={manualPRs.first_olympic?.value} date={manualPRs.first_olympic?.date} manualKey="first_olympic" placeholder="Race-Zeit" />
      </div>

      <div className="text-[10px] text-slate-500 italic mt-2 pt-2 border-t border-slate-100">
        Auto-Werte aus deinen Logs. Tippe ✏️ zum manuellen Eintragen. 1RM nach Epley-Formel geschätzt — am genauesten bei 1–8 Wdh.
      </div>
    </Card>
  );
};

const MehrTab = ({ weightLog, swimLog, strengthLog, dailyLogs, nutritionLogs, cardioTests, manualPRs, setManualPRs }) => {
  const generateReport = () => {
    const phase = getCurrentPhase();
    const phaseNum = phase.num || 1;
    const longestSwim = swimLog.reduce((max, s) => Math.max(max, s.longestContinuous || 0), 0);
    const latestWeight = weightLog.length > 0 ? weightLog[weightLog.length - 1] : null;
    const weightChange = latestWeight ? (latestWeight.weight - 78.2).toFixed(1) : 'kein Eintrag';
    const recent7 = dailyLogs.filter(l => daysBetween(l.date, todayISO()) <= 7);
    const completed = recent7.filter(l => l.completed).length;
    const avgEnergy = recent7.length > 0 ? (recent7.reduce((a, l) => a + (l.energy || 0), 0) / recent7.length).toFixed(1) : '—';
    const avgSleep = recent7.length > 0 ? (recent7.reduce((a, l) => a + (l.sleep || 0), 0) / recent7.length).toFixed(1) : '—';

    let r = `=== TRIATHLON TRACKER REPORT ===\nStand: ${formatDate(todayISO())}\nPhase: ${phase.name}\nTage bis Wettkampf: ${getDaysToRace()}\n\n`;
    r += `KÖRPER:\nGewicht: ${latestWeight?.weight || '—'} kg (Δ Start: ${weightChange} kg)\nEinträge: ${weightLog.length}\n\n`;
    r += `SCHWIMMEN:\nLängste Distanz am Stück: ${longestSwim} m\nEinheiten: ${swimLog.length}\n\n`;
    r += `KRAFT:\nSessions: ${strengthLog.length}\n\n`;
    r += `LETZTE 7 TAGE:\nTrainings: ${completed}/${recent7.length}\nØ Energie: ${avgEnergy}/10\nØ Schlaf: ${avgSleep} h\n\n`;
    r += `CARDIO TESTS: ${cardioTests.length} eingetragen\n\n`;
    r += `LETZTE NOTIZEN:\n`;
    dailyLogs.slice(0, 5).forEach(l => { if (l.notes) r += `${formatDateShort(l.date)}: ${l.notes}\n`; });
    return r;
  };

  const copy = async () => {
    try { await navigator.clipboard.writeText(generateReport()); alert('Report kopiert! Füge ihn in unseren Chat ein.'); }
    catch (e) { alert('Kopieren fehlgeschlagen.'); }
  };

  return (
    <div className="space-y-3">
      <Card>
        <SectionTitle icon={User}>Profil</SectionTitle>
        <div className="grid grid-cols-2 gap-3 text-sm">
          <div><div className="text-xs text-slate-500">Alter</div><div className="font-semibold">21</div></div>
          <div><div className="text-xs text-slate-500">Größe</div><div className="font-semibold">176 cm</div></div>
          <div><div className="text-xs text-slate-500">Startgewicht</div><div className="font-semibold">78,2 kg</div></div>
          <div><div className="text-xs text-slate-500">BMR</div><div className="font-semibold">1.900 kcal</div></div>
        </div>
      </Card>
      <PRsCard swimLog={swimLog} strengthLog={strengthLog} manualPRs={manualPRs} setManualPRs={setManualPRs} />
      <Card>
        <SectionTitle icon={Download}>Export für Coach-Anpassung</SectionTitle>
        <p className="text-xs text-slate-600 mb-3">Kopiere den Report und füge ihn in unseren Claude-Chat ein für tiefere Analyse und Plan-Anpassung.</p>
        <button onClick={copy} className="w-full py-2.5 bg-blue-700 text-white rounded-lg font-semibold flex items-center justify-center gap-2">
          <Download size={16} /> Report kopieren
        </button>
      </Card>
      <Card>
        <SectionTitle icon={Calendar}>Wann zu Claude kommen</SectionTitle>
        <ul className="space-y-1.5 text-sm text-slate-700">
          <li>• <strong>Nach Deload-Wochen</strong> (4, 8, 12): Nächste Phase justieren</li>
          <li>• <strong>Gewicht-Trend ungewöhnlich:</strong> Makros anpassen</li>
          <li>• <strong>Mehrere verpasste Einheiten:</strong> Plan-Anpassung</li>
          <li>• <strong>Schmerzen/Verletzungen:</strong> Sofort, nicht warten</li>
          <li>• <strong>Vor Woche 9:</strong> Freiwasser-Check, Equipment</li>
          <li>• <strong>Vor Renntag:</strong> Strategie, mentale Vorbereitung</li>
        </ul>
      </Card>
      <Card>
        <SectionTitle icon={FileText}>Report-Vorschau</SectionTitle>
        <pre className="text-[10px] bg-slate-50 p-2 rounded-lg overflow-x-auto whitespace-pre-wrap font-mono text-slate-700 max-h-80 overflow-y-auto">
          {generateReport()}
        </pre>
      </Card>
    </div>
  );
};

// ============ MAIN APP ============
function App() {
  const [weightLog, setWeightLog] = useStorage('weight-log', []);
  const [swimLog, setSwimLog] = useStorage('swim-log', []);
  const [strengthLog, setStrengthLog] = useStorage('strength-log', []);
  const [dailyLogs, setDailyLogs] = useStorage('daily-logs', []);
  const [nutritionLogs, setNutritionLogs] = useStorage('nutrition-logs', []);
  const [cardioTests, setCardioTests] = useStorage('cardio-tests', []);
  const [mobilityLogs, setMobilityLogs] = useStorage('mobility-logs', []);
  const [manualPRs, setManualPRs] = useStorage('manual-prs', {});
  const [tab, setTab] = useState('dashboard');

  const tabs = [
    { id: 'dashboard', label: 'Heute', icon: Home },
    { id: 'plan', label: 'Plan', icon: Calendar },
    { id: 'tracker', label: 'Tracker', icon: Activity },
    { id: 'wissen', label: 'Wissen', icon: BookOpen },
    { id: 'mehr', label: 'Mehr', icon: Menu },
  ];

  return (
    <div className="min-h-screen bg-slate-100 pb-20">
      <div className="bg-gradient-to-r from-blue-800 to-blue-600 text-white p-4 sticky top-0 z-10 shadow-md">
        <h1 className="text-lg font-bold">🏊‍♂️ Tri-Tracker</h1>
        <p className="text-[10px] text-blue-100">Olympic · 30.08.2026 · {getDaysToRace()} Tage</p>
      </div>

      <div className="max-w-2xl mx-auto p-3">
        {tab === 'dashboard' && <DashboardTab weightLog={weightLog} swimLog={swimLog} strengthLog={strengthLog} dailyLogs={dailyLogs} goTo={setTab} />}
        {tab === 'plan' && <PlanTab />}
        {tab === 'tracker' && <TrackerTab
          weightLog={weightLog} setWeightLog={setWeightLog}
          swimLog={swimLog} setSwimLog={setSwimLog}
          strengthLog={strengthLog} setStrengthLog={setStrengthLog}
          dailyLogs={dailyLogs} setDailyLogs={setDailyLogs}
          cardioTests={cardioTests} setCardioTests={setCardioTests}
          nutritionLogs={nutritionLogs} setNutritionLogs={setNutritionLogs}
          mobilityLogs={mobilityLogs} setMobilityLogs={setMobilityLogs}
        />}
        {tab === 'wissen' && <WissenTab />}
        {tab === 'mehr' && <MehrTab weightLog={weightLog} swimLog={swimLog} strengthLog={strengthLog} dailyLogs={dailyLogs} nutritionLogs={nutritionLogs} cardioTests={cardioTests} manualPRs={manualPRs} setManualPRs={setManualPRs} />}
      </div>

      <nav className="fixed bottom-0 left-0 right-0 bg-white border-t border-slate-200 shadow-lg">
        <div className="max-w-2xl mx-auto grid grid-cols-5">
          {tabs.map(t => {
            const Icon = t.icon;
            const active = tab === t.id;
            return (
              <button key={t.id} onClick={() => setTab(t.id)}
                className={`flex flex-col items-center gap-0.5 py-2.5 relative ${active ? 'text-blue-700' : 'text-slate-500'}`}>
                <Icon size={20} />
                <span className="text-[10px] font-medium">{t.label}</span>
                {active && <div className="absolute top-0 w-12 h-0.5 bg-blue-700 rounded-b" />}
              </button>
            );
          })}
        </div>
      </nav>
    </div>
  );
}


// === RENDER ===
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(<App />);
// Hide loading screen once mounted
setTimeout(() => {
  const l = document.getElementById('loading');
  if (l) l.classList.add('hidden');
}, 100);

</script>
</body>
</html>
