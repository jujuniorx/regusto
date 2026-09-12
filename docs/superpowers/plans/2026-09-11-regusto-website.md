# Regusto Website Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the Regusto restaurant marketing site — single-page, institutional + cardápio + pedido via WhatsApp — per the approved design spec.

**Architecture:** Next.js (App Router, TypeScript, `src/` dir) + Tailwind CSS v4 + GSAP/ScrollTrigger. Content lives in typed files under `src/content/`, not a CMS. One page (`/`) composed of section components, each independently built and reveal-animated. No backend, no database — the only "dynamic" behavior is a `wa.me` deep link.

**Tech Stack:** Next.js (latest stable, App Router), TypeScript, Tailwind CSS v4 (`@theme` tokens, no `tailwind.config.js`), GSAP + `@gsap/react` (`useGSAP`, ScrollTrigger), `next/font/google` (Fraunces, Work Sans), deploy target Vercel.

**Spec:** `docs/superpowers/specs/2026-09-11-regusto-website-design.md` (and companion `docs/superpowers/specs/2026-09-11-regusto-design-dna.json`)

## Global Constraints

- Node.js 20 LTS or newer, npm as package manager.
- Import alias `@/*` → `src/*` (set during scaffold).
- Colors (Tailwind tokens, exact hex from spec): `primary #A8432B`, `secondary #2F3B2E`, `accent #E8A33D`, `neutral-100 #FBF6EE` … `neutral-700 #1E1A16`, `surface-background #FBF6EE`, `surface-card #FFFDF8`, `surface-elevated #F1E7D8`.
- Fonts: **Fraunces** (`font-heading`) for headings/display, **Work Sans** (`font-sans`) for body/UI/prices, both via `next/font/google`.
- Radii: `sm 4px`, `md 10px`, `lg 20px`, `pill 999px`.
- Motion: GSAP + ScrollTrigger (+ `SplitText`, added in Task 5 — part of the `gsap` package, no separate install) only — no Framer Motion, no WebGL/Three.js/particles/shaders/custom cursor (explicitly out of scope per spec §9, unless an exceptional case justifies it during implementation — never forced). Every animation must be wrapped so `prefers-reduced-motion: reduce` disables movement (`gsap.matchMedia()`), per spec §7 — with no exception, including the Hero and Como Funciona sections below. The spec's "ease-out orgânico" cubic-bezier(0.22,1,0.36,1) is implemented with GSAP's built-in `"power3.out"` (no CustomEase plugin — closest stock ease, avoids an unneeded dependency).
- **Motion hierarchy (spec §11.3, permanent rule):** Hero = impact (2-layer parallax + text reveal on load). Como Funciona = the site's one big interactive moment — the only other section allowed to use scroll-scrub/pin. Sobre/Ambiente = subtle storytelling, one-shot fade-up only, no scrub. Cardápio = clarity/conversion, no motion flourish. Contato = action-focused, no decorative motion. No task in this plan may add scrub/pin to a section other than Hero (Task 8) or Como Funciona (Task 10) without updating this rule first.
- 21st.dev was used only as a curated research/reference step before this plan (spec §11) — no component from it is installed or copied verbatim anywhere in this plan. Where a task below cites a 21st pattern (Editorial Image Hero, Horizontal Feature Reveal), it is rebuilt from scratch with this project's own tokens, content, and GSAP stack.
- Content: no hardcoded business facts inside components. Everything (name, tagline, address, phone, WhatsApp number, hours, payment methods, CNPJ, menu items/prices) comes from `src/content/site.ts` / `src/content/menu.ts`.
- Business hours: single source of truth is `siteContent.businessHours` (spec §2.4, §6) — never duplicate the hours string in a component.
- No automated unit test framework for this MVP (spec §8 — content/marketing site, no business logic). Each task's verification is `npm run build` (type-check + lint, since Next.js runs ESLint during build) plus a concrete manual browser check described in the task.
- No CMS, no reservation form/backend, no multi-language, no `/cardapio` route (spec §9).
- Any field whose real value is still pending from the client (spec §6: menu item names/prices beyond confirmed categories, "Sobre" text, real photos) ships as clearly-marked placeholder content — `isPlaceholder: true` on menu items, `PhotoPlaceholder` component for images, an inline `{/* PLACEHOLDER: ... */}` comment for prose — never silently invented as if real.

---

### Task 1: Scaffold the Next.js project

**Files:**
- Create: entire Next.js scaffold at repo root (`package.json`, `next.config.ts`, `tsconfig.json`, `eslint.config.mjs`, `postcss.config.mjs`, `src/app/layout.tsx`, `src/app/page.tsx`, `src/app/globals.css`, `public/*`)

**Interfaces:**
- Consumes: nothing (first task)
- Produces: a working Next.js dev/build environment with `src/` dir and `@/*` import alias, which every later task relies on

- [ ] **Step 1: Scaffold into a temp directory (repo root already has `.git` and `docs/`, so `create-next-app` can't target `.` directly)**

Run:
```bash
npx create-next-app@latest _scaffold_tmp --yes --src-dir --disable-git
```

- [ ] **Step 2: Merge the scaffold into the repo root**

Run:
```bash
cd _scaffold_tmp
find . -mindepth 1 -maxdepth 1 -exec mv {} .. \;
cd ..
rmdir _scaffold_tmp
```

- [ ] **Step 3: Verify the dev server boots**

Run: `npm run dev` (then stop it, e.g. Ctrl+C, once confirmed) — or `npm run build` for a non-interactive check.
Expected: build succeeds with no errors; default Next.js starter page compiles.

- [ ] **Step 4: Confirm git sees the regenerated scaffold as modifications, not new untracked chaos**

Run: `git status`
Expected: the previously-`deleted` scaffold files (package.json, tsconfig.json, etc.) now show as `modified`; no leftover `_scaffold_tmp` directory.

- [ ] **Step 5: Commit**

```bash
git add package.json package-lock.json next.config.ts tsconfig.json eslint.config.mjs postcss.config.mjs public src AGENTS.md CLAUDE.md .gitignore README.md
git commit -m "chore: scaffold Next.js app (App Router, TS, Tailwind, src dir)"
```

---

### Task 2: Design tokens — Tailwind v4 theme + fonts

**Files:**
- Modify: `src/app/layout.tsx`
- Modify: `src/app/globals.css`

**Interfaces:**
- Consumes: the scaffold from Task 1
- Produces: Tailwind utilities `bg-primary`, `bg-secondary`, `bg-accent`, `bg-neutral-100`..`bg-neutral-700` (+ `text-*`, `border-*` variants), `bg-surface-background`, `bg-surface-card`, `bg-surface-elevated`, `rounded-sm|md|lg|pill`, `font-heading`, `font-sans` — used by every component task from here on

- [ ] **Step 1: Wire fonts in the root layout**

```tsx
// src/app/layout.tsx
import type { Metadata } from "next";
import { Fraunces, Work_Sans } from "next/font/google";
import "./globals.css";

const fraunces = Fraunces({
  subsets: ["latin"],
  display: "swap",
  variable: "--font-fraunces",
  weight: ["400", "500", "600"],
});

const workSans = Work_Sans({
  subsets: ["latin"],
  display: "swap",
  variable: "--font-work-sans",
  weight: ["400", "500", "600"],
});

export const metadata: Metadata = {
  title: "Regusto — Comida caseira de qualidade",
  description:
    "Restaurante self-service por quilo, marmitex e grelhados no Jardim América, Bauru/SP.",
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="pt-BR" className={`${fraunces.variable} ${workSans.variable}`}>
      <body>{children}</body>
    </html>
  );
}
```

- [ ] **Step 2: Replace the default Tailwind theme with Regusto's tokens**

```css
/* src/app/globals.css */
@import "tailwindcss";

@theme {
  --color-primary: #a8432b;
  --color-secondary: #2f3b2e;
  --color-accent: #e8a33d;

  --color-neutral-100: #fbf6ee;
  --color-neutral-200: #f1e7d8;
  --color-neutral-300: #ddd0bc;
  --color-neutral-400: #a99c88;
  --color-neutral-500: #6b6154;
  --color-neutral-600: #3a342c;
  --color-neutral-700: #1e1a16;

  --color-surface-background: #fbf6ee;
  --color-surface-card: #fffdf8;
  --color-surface-elevated: #f1e7d8;

  --radius-sm: 4px;
  --radius-md: 10px;
  --radius-lg: 20px;
  --radius-pill: 999px;
}

@theme inline {
  --font-heading: var(--font-fraunces);
  --font-sans: var(--font-work-sans);
}

body {
  background-color: var(--color-surface-background);
  color: var(--color-neutral-700);
  font-family: var(--font-sans);
}
```

- [ ] **Step 3: Verify build and visually confirm tokens are live**

Run: `npm run build`
Expected: no errors.

Then run `npm run dev`, open `http://localhost:3000`, temporarily add `className="bg-primary text-neutral-100 p-4 font-heading"` to any element on the default page, confirm it renders terracotta background, cream text, and the serif (Fraunces) font — then remove the temporary className before committing.

- [ ] **Step 4: Commit**

```bash
git add src/app/layout.tsx src/app/globals.css
git commit -m "feat: add Regusto design tokens and fonts to Tailwind theme"
```

---

### Task 3: Content layer — site data, menu data, WhatsApp URL builder

**Files:**
- Create: `src/content/site.ts`
- Create: `src/content/menu.ts`
- Create: `src/lib/whatsapp.ts`

**Interfaces:**
- Consumes: nothing beyond TypeScript
- Produces:
  - `siteContent: SiteContent` (from `@/content/site`) with fields `brandName, tagline, description, phone, whatsappNumber, whatsappDefaultMessage, address: {street, neighborhood, city, state, zip}, businessHours: {display, source, confirmedForLaunch}, paymentMethods: string[], cnpj, instagramUrl, mapsEmbedUrl`
  - `menuCategories: MenuCategory[]` and `formatPriceBRL(price: number): string` (from `@/content/menu`), where `MenuCategory = {id, name, items: MenuItem[]}` and `MenuItem = {name, description?, price, isPlaceholder}`
  - `buildWhatsAppOrderUrl(phoneDigits: string, message: string): string` (from `@/lib/whatsapp`)

- [ ] **Step 1: Write `src/content/site.ts`**

```ts
export interface BusinessHours {
  display: string;
  source: string;
  confirmedForLaunch: boolean;
}

export interface SiteAddress {
  street: string;
  neighborhood: string;
  city: string;
  state: string;
  zip: string;
}

export interface SiteContent {
  brandName: string;
  tagline: string;
  description: string;
  phone: string;
  whatsappNumber: string;
  whatsappDefaultMessage: string;
  address: SiteAddress;
  businessHours: BusinessHours;
  paymentMethods: string[];
  cnpj: string;
  instagramUrl: string;
  mapsEmbedUrl: string;
}

export const siteContent: SiteContent = {
  brandName: "Regusto",
  tagline: "Comida caseira de qualidade",
  description:
    "Self-service por quilo, marmitex e grelhados no Jardim América, Bauru/SP — o almoço de todo dia, feito com cuidado de casa.",
  phone: "(14) 3204-4151",
  whatsappNumber: "5511998731881",
  whatsappDefaultMessage: "Olá! Gostaria de fazer um pedido na Regusto.",
  address: {
    street: "Av. Getúlio Vargas, 16-70",
    neighborhood: "Jardim América",
    city: "Bauru",
    state: "SP",
    zip: "17017-339",
  },
  businessHours: {
    display: "Segunda a sábado: 08:00–14:20 · Domingo: fechado",
    source: "site oficial (regusto.com.br)",
    // Diverge de agregadores externos (~11:00–15:00) — ver spec §2.4.
    // NÃO substituir automaticamente. Confirmar com o cliente antes do
    // lançamento e só então marcar como true.
    confirmedForLaunch: false,
  },
  paymentMethods: [
    "Dinheiro",
    "PIX",
    "Cartão de crédito (Visa, Mastercard, Hipercard, Elo)",
    "Cartão de débito",
    "Vale-refeição (Alelo, Verocad, Sodexo, Ticket Restaurante)",
  ],
  cnpj: "27.001.306/0001-19",
  instagramUrl: "https://www.instagram.com/regustorestaurante/",
  mapsEmbedUrl:
    "https://www.google.com/maps?q=Av.+Get%C3%BAlio+Vargas,+16-70,+Jardim+Am%C3%A9rica,+Bauru+-+SP,+17017-339&output=embed",
};
```

- [ ] **Step 2: Write `src/content/menu.ts`**

```ts
export interface MenuItem {
  name: string;
  description?: string;
  price: number;
  isPlaceholder: boolean;
}

export interface MenuCategory {
  id: string;
  name: string;
  items: MenuItem[];
}

// Categorias confirmadas via pesquisa (spec §2.1). Itens e preços são
// placeholders ilustrativos até o cliente enviar o cardápio real (spec §6).
export const menuCategories: MenuCategory[] = [
  {
    id: "grelhados",
    name: "Grelhados à la Carte",
    items: [
      {
        name: "Picanha grelhada",
        description: "Acompanha arroz, farofa e vinagrete",
        price: 42.9,
        isPlaceholder: true,
      },
      {
        name: "Frango grelhado",
        description: "Acompanha arroz e salada",
        price: 28.9,
        isPlaceholder: true,
      },
    ],
  },
  {
    id: "marmitex",
    name: "Marmitex",
    items: [
      {
        name: "Marmitex tradicional",
        description: "Proteína do dia + 2 acompanhamentos",
        price: 24.9,
        isPlaceholder: true,
      },
    ],
  },
  {
    id: "pasteis",
    name: "Pastéis",
    items: [
      { name: "Pastel de carne", price: 9.9, isPlaceholder: true },
      { name: "Pastel de queijo", price: 9.9, isPlaceholder: true },
    ],
  },
  {
    id: "saladas",
    name: "Saladas",
    items: [{ name: "Salada mista", price: 12.9, isPlaceholder: true }],
  },
  {
    id: "bebidas",
    name: "Bebidas",
    items: [{ name: "Suco natural", price: 8.9, isPlaceholder: true }],
  },
];

export function formatPriceBRL(price: number): string {
  return price.toLocaleString("pt-BR", { style: "currency", currency: "BRL" });
}
```

- [ ] **Step 3: Write `src/lib/whatsapp.ts`**

```ts
export function buildWhatsAppOrderUrl(phoneDigits: string, message: string): string {
  return `https://wa.me/${phoneDigits}?text=${encodeURIComponent(message)}`;
}
```

- [ ] **Step 4: Verify with a type-check build**

Run: `npm run build`
Expected: no TypeScript errors.

- [ ] **Step 5: Commit**

```bash
git add src/content src/lib
git commit -m "feat: add typed content layer for site data and menu"
```

---

### Task 4: UI primitives

**Files:**
- Create: `src/components/ui/Button.tsx`
- Create: `src/components/ui/Container.tsx`
- Create: `src/components/ui/Section.tsx`
- Create: `src/components/ui/SectionHeading.tsx`
- Create: `src/components/ui/PhotoPlaceholder.tsx`
- Create: `src/components/ui/icons.tsx`

**Interfaces:**
- Consumes: Tailwind tokens from Task 2
- Produces: `Button({variant?: "primary"|"ghost", href?, children, className?, ...rest})`, `Container({children, className?})`, `Section({id?, className?, children})`, `SectionHeading({overline?, title, align?: "left"|"center"})`, `PhotoPlaceholder({label, aspectClassName?, roundedClassName?, className?})`, and icons `MapPinIcon`, `ClockIcon`, `MenuIcon`, `CloseIcon`, `WhatsAppIcon` (all `(props: SVGProps<SVGSVGElement>) => JSX.Element`) — used by every section task from here on

- [ ] **Step 1: `src/components/ui/Button.tsx`**

```tsx
import type { AnchorHTMLAttributes, ButtonHTMLAttributes, ReactNode } from "react";

export type ButtonVariant = "primary" | "ghost";

interface BaseProps {
  variant?: ButtonVariant;
  children: ReactNode;
  className?: string;
}

type ButtonAsButton = BaseProps & ButtonHTMLAttributes<HTMLButtonElement> & { href?: undefined };
type ButtonAsLink = BaseProps & AnchorHTMLAttributes<HTMLAnchorElement> & { href: string };

export type ButtonProps = ButtonAsButton | ButtonAsLink;

const VARIANT_CLASSES: Record<ButtonVariant, string> = {
  primary: "bg-primary text-neutral-100 hover:opacity-90",
  ghost: "border border-neutral-400 text-neutral-700 hover:bg-neutral-200",
};

export function Button({ variant = "primary", children, className, ...rest }: ButtonProps) {
  const classes = `inline-flex items-center justify-center rounded-md px-6 py-3 font-sans font-semibold transition-colors duration-200 ${VARIANT_CLASSES[variant]} ${className ?? ""}`;

  if ("href" in rest && rest.href) {
    return (
      <a className={classes} {...(rest as AnchorHTMLAttributes<HTMLAnchorElement>)}>
        {children}
      </a>
    );
  }

  return (
    <button className={classes} {...(rest as ButtonHTMLAttributes<HTMLButtonElement>)}>
      {children}
    </button>
  );
}
```

- [ ] **Step 2: `src/components/ui/Container.tsx` and `src/components/ui/Section.tsx`**

```tsx
// src/components/ui/Container.tsx
import type { ReactNode } from "react";

export function Container({ children, className }: { children: ReactNode; className?: string }) {
  return <div className={`mx-auto w-full max-w-[1200px] px-6 sm:px-10 ${className ?? ""}`}>{children}</div>;
}
```

```tsx
// src/components/ui/Section.tsx
import type { ReactNode } from "react";

export interface SectionProps {
  id?: string;
  className?: string;
  children: ReactNode;
}

export function Section({ id, className, children }: SectionProps) {
  return (
    <section id={id} className={`py-24 ${className ?? ""}`}>
      {children}
    </section>
  );
}
```

- [ ] **Step 3: `src/components/ui/SectionHeading.tsx` and `src/components/ui/PhotoPlaceholder.tsx`**

```tsx
// src/components/ui/SectionHeading.tsx
export interface SectionHeadingProps {
  overline?: string;
  title: string;
  align?: "left" | "center";
}

export function SectionHeading({ overline, title, align = "left" }: SectionHeadingProps) {
  return (
    <div className={align === "center" ? "text-center" : "text-left"}>
      {overline && (
        <p className="mb-2 text-xs font-semibold uppercase tracking-[0.12em] text-primary">{overline}</p>
      )}
      <h2 className="font-heading text-[clamp(1.5rem,2.6vw,2.25rem)] font-medium leading-[1.15] text-neutral-700">
        {title}
      </h2>
    </div>
  );
}
```

```tsx
// src/components/ui/PhotoPlaceholder.tsx
export interface PhotoPlaceholderProps {
  label: string;
  aspectClassName?: string;
  roundedClassName?: string;
  className?: string;
}

// Usado onde uma foto real ainda não foi recebida do cliente (spec §6).
// Substituir por <Image> real quando os arquivos chegarem.
// roundedClassName is its own prop (not folded into className) because Tailwind's
// generated stylesheet orders radius utilities by scale, not by className string
// order — passing "rounded-none" in className would not reliably beat a hardcoded
// "rounded-lg". Callers that need a square-cornered, full-bleed placeholder (e.g.
// the Hero) pass roundedClassName="" instead.
export function PhotoPlaceholder({
  label,
  aspectClassName = "aspect-[4/5]",
  roundedClassName = "rounded-lg",
  className,
}: PhotoPlaceholderProps) {
  return (
    <div
      role="img"
      aria-label={label}
      className={`flex items-center justify-center bg-neutral-200 text-center text-sm text-neutral-500 ${roundedClassName} ${aspectClassName} ${className ?? ""}`}
    >
      <span className="px-4">{label}</span>
    </div>
  );
}
```

- [ ] **Step 4: `src/components/ui/icons.tsx`**

```tsx
import type { SVGProps } from "react";

function BaseIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <svg
      viewBox="0 0 24 24"
      fill="none"
      stroke="currentColor"
      strokeWidth={1.5}
      strokeLinecap="round"
      strokeLinejoin="round"
      aria-hidden="true"
      {...props}
    />
  );
}

export function MapPinIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <BaseIcon {...props}>
      <path d="M12 21s-7-6.1-7-11a7 7 0 0 1 14 0c0 4.9-7 11-7 11Z" />
      <circle cx="12" cy="10" r="2.5" />
    </BaseIcon>
  );
}

export function ClockIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <BaseIcon {...props}>
      <circle cx="12" cy="12" r="9" />
      <path d="M12 7v5l3.5 2" />
    </BaseIcon>
  );
}

export function MenuIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <BaseIcon {...props}>
      <path d="M4 7h16M4 12h16M4 17h16" />
    </BaseIcon>
  );
}

export function CloseIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <BaseIcon {...props}>
      <path d="M6 6l12 12M18 6 6 18" />
    </BaseIcon>
  );
}

export function WhatsAppIcon(props: SVGProps<SVGSVGElement>) {
  return (
    <BaseIcon {...props}>
      <path d="M7 17l-1.4 3.2a.5.5 0 0 0 .66.66L9.5 19.5A8.5 8.5 0 1 0 4 14.5c0 .9.16 1.76.46 2.55Z" />
      <path d="M9 10.5c0 3 2.5 5.5 5.5 5.5" />
    </BaseIcon>
  );
}
```

- [ ] **Step 5: Verify and commit**

Run: `npm run build` — expect no errors.

```bash
git add src/components/ui
git commit -m "feat: add shared UI primitives (Button, Container, Section, icons)"
```

---

### Task 5: GSAP setup and the `Reveal` scroll primitive

**Files:**
- Create: `src/lib/gsap.ts`
- Create: `src/components/ui/Reveal.tsx`

**Interfaces:**
- Consumes: nothing new (pure infra)
- Produces: `gsap`, `ScrollTrigger`, `SplitText`, `useGSAP` re-exported from `@/lib/gsap` (plugin registration happens once, here); `Reveal({children, className?, delay?})` (from `@/components/ui/Reveal`) — the fade-up-on-scroll wrapper reused by most sections

- [ ] **Step 1: Install GSAP**

Run:
```bash
npm install gsap @gsap/react
```

- [ ] **Step 2: Verify the current `SplitText` import path via context7 before wiring it up**

`SplitText` is a GSAP plugin (free in the `gsap` package for all users since the GreenSock/Webflow licensing change — no Club GreenSock registry, no extra install beyond Step 1). Package APIs shift between versions, so before writing the registration below, query context7 for the installed `gsap` version's current `SplitText` import path and registration pattern (it has moved at least once, e.g. `gsap/SplitText` vs. a scoped export) and use whatever it confirms instead of assuming the snippet here is still exact.

- [ ] **Step 3: Centralize plugin registration in `src/lib/gsap.ts`**

```ts
"use client";

import gsap from "gsap";
import { ScrollTrigger } from "gsap/ScrollTrigger";
import { SplitText } from "gsap/SplitText";
import { useGSAP } from "@gsap/react";

gsap.registerPlugin(ScrollTrigger, SplitText, useGSAP);

export { gsap, ScrollTrigger, SplitText, useGSAP };
```

- [ ] **Step 4: Build the `Reveal` primitive**

```tsx
// src/components/ui/Reveal.tsx
"use client";

import { useRef, type ReactNode } from "react";
import { gsap, useGSAP } from "@/lib/gsap";

export interface RevealProps {
  children: ReactNode;
  className?: string;
  delay?: number;
}

export function Reveal({ children, className, delay = 0 }: RevealProps) {
  const ref = useRef<HTMLDivElement>(null);

  useGSAP(
    () => {
      const mm = gsap.matchMedia();

      mm.add(
        { motionReduced: "(prefers-reduced-motion: reduce)" },
        (context) => {
          const { motionReduced } = context.conditions as { motionReduced: boolean };

          gsap.from(ref.current, {
            autoAlpha: 0,
            y: motionReduced ? 0 : 24,
            duration: motionReduced ? 0.01 : 0.6,
            delay,
            ease: "power3.out",
            scrollTrigger: {
              trigger: ref.current,
              start: "top 85%",
              toggleActions: "play none none none",
            },
          });
        }
      );

      return () => mm.revert();
    },
    { scope: ref }
  );

  return (
    <div ref={ref} className={className}>
      {children}
    </div>
  );
}
```

- [ ] **Step 5: Verify with a throwaway visual check**

Temporarily wrap the default homepage's `<h1>` in `<Reveal>` (import from `@/components/ui/Reveal`), run `npm run dev`, reload `http://localhost:3000`, and confirm the heading fades/slides up shortly after load. Then revert that temporary edit (do not commit it) — this task ships only the primitive, not a page using it yet.

Run: `npm run build` — expect no errors.

- [ ] **Step 6: Commit**

```bash
git add package.json package-lock.json src/lib/gsap.ts src/components/ui/Reveal.tsx
git commit -m "feat: install GSAP (+ SplitText) and add Reveal scroll-in primitive"
```

---

### Task 6: WhatsApp order button

**Files:**
- Create: `src/components/WhatsAppOrderButton.tsx`

**Interfaces:**
- Consumes: `siteContent` (`@/content/site`), `buildWhatsAppOrderUrl` (`@/lib/whatsapp`), `Button` and `WhatsAppIcon` (`@/components/ui/*`)
- Produces: `WhatsAppOrderButton({label?, className?})` — the reusable order CTA used by Header, Hero, Menu, and Footer

- [ ] **Step 1: Write the component**

```tsx
// src/components/WhatsAppOrderButton.tsx
import { siteContent } from "@/content/site";
import { buildWhatsAppOrderUrl } from "@/lib/whatsapp";
import { Button } from "@/components/ui/Button";
import { WhatsAppIcon } from "@/components/ui/icons";

export interface WhatsAppOrderButtonProps {
  label?: string;
  className?: string;
}

export function WhatsAppOrderButton({ label = "Peça pelo WhatsApp", className }: WhatsAppOrderButtonProps) {
  const href = buildWhatsAppOrderUrl(siteContent.whatsappNumber, siteContent.whatsappDefaultMessage);

  return (
    <Button href={href} target="_blank" rel="noopener noreferrer" className={className}>
      <WhatsAppIcon className="mr-2 h-5 w-5" />
      {label}
    </Button>
  );
}
```

- [ ] **Step 2: Verify manually**

Temporarily render `<WhatsAppOrderButton />` on the homepage, run `npm run dev`, click it, and confirm it opens `https://wa.me/5511998731881?text=...` with the message URL-decoded to "Olá! Gostaria de fazer um pedido na Regusto." in a new tab. Revert the temporary render (not committed).

Run: `npm run build` — expect no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/WhatsAppOrderButton.tsx
git commit -m "feat: add reusable WhatsApp order button"
```

---

### Task 7: Header / navigation

**Files:**
- Create: `src/components/Header.tsx`

**Interfaces:**
- Consumes: `gsap`, `ScrollTrigger`, `useGSAP` (`@/lib/gsap`), `siteContent` (`@/content/site`), `WhatsAppOrderButton`, `MenuIcon`, `CloseIcon`
- Produces: `Header()` — fixed nav, rendered first in `page.tsx` (Task 14). Anchor targets it links to (`#sobre`, `#como-funciona`, `#cardapio`, `#ambiente`, `#informacoes`, `#contato`) are the section `id`s that Tasks 9–13 must use verbatim.

- [ ] **Step 1: Write the component**

```tsx
// src/components/Header.tsx
"use client";

import { useRef, useState } from "react";
import { gsap, ScrollTrigger, useGSAP } from "@/lib/gsap";
import { siteContent } from "@/content/site";
import { WhatsAppOrderButton } from "@/components/WhatsAppOrderButton";
import { MenuIcon, CloseIcon } from "@/components/ui/icons";

const NAV_ITEMS = [
  { href: "#sobre", label: "Sobre" },
  { href: "#como-funciona", label: "Como Funciona" },
  { href: "#cardapio", label: "Cardápio" },
  { href: "#ambiente", label: "Ambiente" },
  { href: "#informacoes", label: "Informações" },
  { href: "#contato", label: "Contato" },
];

export function Header() {
  const headerRef = useRef<HTMLElement>(null);
  const [mobileOpen, setMobileOpen] = useState(false);

  useGSAP(
    () => {
      const trigger = ScrollTrigger.create({
        start: "top -80",
        onToggle: (self) => {
          headerRef.current?.classList.toggle("bg-neutral-100", self.isActive);
          headerRef.current?.classList.toggle("shadow-sm", self.isActive);
        },
      });

      return () => trigger.kill();
    },
    { scope: headerRef }
  );

  return (
    <header
      ref={headerRef}
      className="fixed inset-x-0 top-0 z-50 flex items-center justify-between px-6 py-4 transition-colors duration-300 sm:px-10"
    >
      <a href="#hero" className="font-heading text-xl font-medium text-neutral-700">
        {siteContent.brandName}
      </a>

      <nav className="hidden gap-6 md:flex">
        {NAV_ITEMS.map((item) => (
          <a key={item.href} href={item.href} className="text-sm font-medium text-neutral-700 hover:text-primary">
            {item.label}
          </a>
        ))}
      </nav>

      <div className="hidden md:block">
        <WhatsAppOrderButton label="Pedir" />
      </div>

      <button
        type="button"
        className="md:hidden"
        aria-label={mobileOpen ? "Fechar menu" : "Abrir menu"}
        onClick={() => setMobileOpen((open) => !open)}
      >
        {mobileOpen ? <CloseIcon className="h-6 w-6" /> : <MenuIcon className="h-6 w-6" />}
      </button>

      {mobileOpen && (
        <div className="fixed inset-0 top-[64px] flex flex-col items-center gap-8 bg-neutral-100 pt-12 md:hidden">
          {NAV_ITEMS.map((item) => (
            <a
              key={item.href}
              href={item.href}
              className="font-heading text-2xl text-neutral-700"
              onClick={() => setMobileOpen(false)}
            >
              {item.label}
            </a>
          ))}
          <WhatsAppOrderButton />
        </div>
      )}
    </header>
  );
}
```

- [ ] **Step 2: Verify manually**

Temporarily render `<Header />` alone on the homepage (`@/components/Header`), run `npm run dev`. Confirm: header is transparent at the top, gains a cream background + shadow after scrolling ~80px, desktop nav links are visible ≥768px width, and at <768px the hamburger opens a full-screen overlay menu that closes on link click. Revert the temporary render (Task 14 wires it in permanently).

Run: `npm run build` — expect no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/Header.tsx
git commit -m "feat: add site header with scroll-aware background and mobile menu"
```

---

### Task 8: Hero section — full-bleed photography, editorial reveal

Per spec §11.1: the hero must make photography the protagonist (it did not before this revision — the original draft had no image at all, just a solid `bg-secondary` fill). Structural inspiration only from the 21st.dev "Editorial Image Hero" reference (not copied): full-bleed photo, elegant fade, tagline, large display type, CTA. Rebuilt here with this project's own tokens, real content, and the word-reveal mechanic already validated for this project — now implemented with real GSAP `SplitText` instead of manual string-splitting, per spec §11.1 and the Design DNA's `text_effects.technology`.

**Files:**
- Create: `src/components/Hero.tsx`

**Interfaces:**
- Consumes: `gsap`, `SplitText`, `useGSAP` (`@/lib/gsap`), `siteContent`, `WhatsAppOrderButton`, `PhotoPlaceholder` (`@/components/ui/PhotoPlaceholder`)
- Produces: `Hero()` with `id="hero"` (the header's logo link targets `#hero`)

- [ ] **Step 1: Write the component**

```tsx
// src/components/Hero.tsx
"use client";

import { useRef } from "react";
import { gsap, SplitText, useGSAP } from "@/lib/gsap";
import { siteContent } from "@/content/site";
import { WhatsAppOrderButton } from "@/components/WhatsAppOrderButton";
import { PhotoPlaceholder } from "@/components/ui/PhotoPlaceholder";

export function Hero() {
  const scopeRef = useRef<HTMLDivElement>(null);
  const headlineRef = useRef<HTMLHeadingElement>(null);

  useGSAP(
    () => {
      const mm = gsap.matchMedia();

      mm.add({ motionReduced: "(prefers-reduced-motion: reduce)" }, (context) => {
        const { motionReduced } = context.conditions as { motionReduced: boolean };

        gsap.set(".hero-photo", { autoAlpha: motionReduced ? 1 : 0, scale: motionReduced ? 1 : 1.04 });
        gsap.set(".hero-signature-path", {
          strokeDasharray: 300,
          strokeDashoffset: motionReduced ? 0 : 300,
        });

        // SplitText needs the real Fraunces text laid out before it can measure
        // lines — next/font has already loaded it by the time this effect runs.
        const split = headlineRef.current
          ? new SplitText(headlineRef.current, { type: "lines", mask: "lines" })
          : null;
        if (split) gsap.set(split.lines, { yPercent: motionReduced ? 0 : 110 });

        if (!motionReduced) {
          const tl = gsap.timeline({ delay: 0.15 });
          tl.to(".hero-photo", { autoAlpha: 1, scale: 1, duration: 1.1, ease: "power2.out" })
            .to(
              split ? split.lines : [],
              { yPercent: 0, duration: 0.7, ease: "power3.out", stagger: 0.06 },
              "-=0.7"
            )
            .to(".hero-signature-path", { strokeDashoffset: 0, duration: 0.9, ease: "power2.inOut" }, "-=0.3");
        }

        return () => split?.revert();
      });

      return () => mm.revert();
    },
    { scope: scopeRef }
  );

  return (
    <section
      ref={scopeRef}
      id="hero"
      className="relative flex min-h-[92vh] flex-col justify-end overflow-hidden px-6 pb-16 pt-32 text-neutral-100 sm:px-10"
    >
      {/* PLACEHOLDER: swap for a real prato/ambiente photo (next/image) the moment
          the client sends one — spec §6/§11.1. Layout does not change either way. */}
      <div className="hero-photo absolute inset-0">
        <PhotoPlaceholder
          label="Foto de prato ou ambiente da Regusto em destaque"
          aspectClassName="h-full"
          roundedClassName=""
          className="h-full w-full"
        />
      </div>
      <div className="absolute inset-0 bg-gradient-to-t from-secondary via-secondary/55 to-secondary/10" />

      <div className="relative z-10">
        <svg
          className="hero-signature-path mb-6 h-10 w-10 text-accent"
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          strokeWidth={1.5}
          aria-hidden="true"
        >
          <path d="M6 2v8a2 2 0 0 0 2 2v10M10 2v6M14 2v6M14 2a3 3 0 0 1 3 3v5a2 2 0 0 1-2 2v10" />
        </svg>

        {/* Quiet locational line — real siteContent.address fields, not a new
            invented tagline. Sentence case on purpose (spec §11.1: no decorative
            all-caps eyebrow). */}
        <p className="text-[0.8125rem] font-medium tracking-[0.01em] text-neutral-200">
          {siteContent.address.neighborhood}, {siteContent.address.city} — {siteContent.address.state}
        </p>

        <h1
          ref={headlineRef}
          className="mt-3 max-w-3xl font-heading text-[clamp(2.75rem,6vw,5.5rem)] font-medium leading-[1.02] tracking-[-0.01em]"
        >
          {siteContent.tagline}
        </h1>

        <p className="mt-6 max-w-xl text-lg text-neutral-200">{siteContent.description}</p>

        <div className="mt-8">
          <WhatsAppOrderButton />
        </div>
      </div>
    </section>
  );
}
```

- [ ] **Step 2: Verify manually**

Temporarily render `<Hero />` alone on the homepage, run `npm run dev`. Confirm: on load, the placeholder photo fades/scales gently into place, the headline reveals via a line-mask (not a visible flash of unsplit text before the effect attaches), the small signature icon draws itself in, the locational line reads the real neighborhood/city/state, and the WhatsApp CTA is visible and clickable. Then enable "reduce motion" (OS setting or devtools rendering emulation), reload, and confirm the photo, headline, and icon all appear instantly at their final state with no animation. Revert the temporary render.

Run: `npm run build` — expect no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/Hero.tsx src/components/ui/PhotoPlaceholder.tsx
git commit -m "feat: add hero section with full-bleed photo and SplitText headline reveal"
```

---

### Task 9: About section

**Files:**
- Create: `src/components/About.tsx`

**Interfaces:**
- Consumes: `Section`, `Container`, `SectionHeading`, `PhotoPlaceholder`, `Reveal` (all `@/components/ui/*`)
- Produces: `About()` with `id="sobre"`

Motion for this section stays deliberately quiet (spec §11.3: "Sobre/Ambiente → storytelling sutil") — plain `Reveal` fade-up, no scrub, no sticky media. That budget is spent in Task 10 instead.

- [ ] **Step 1: `src/components/About.tsx`**

```tsx
import { Section } from "@/components/ui/Section";
import { Container } from "@/components/ui/Container";
import { SectionHeading } from "@/components/ui/SectionHeading";
import { PhotoPlaceholder } from "@/components/ui/PhotoPlaceholder";
import { Reveal } from "@/components/ui/Reveal";

export function About() {
  return (
    <Section id="sobre">
      <Container className="grid gap-10 md:grid-cols-2 md:items-center">
        <Reveal>
          <SectionHeading overline="Nossa história" title="Comida de casa, feita todo dia" />
          {/* PLACEHOLDER: texto real de história/conceito pendente do cliente (spec §6). */}
          <p className="mt-4 text-neutral-600">
            A Regusto nasceu no Jardim América, em Bauru, com um propósito simples: servir o
            almoço que a gente gostaria de comer em casa — feito na hora, com ingredientes de
            verdade e o cuidado de quem cozinha para quem gosta.
          </p>
        </Reveal>
        <Reveal delay={0.1}>
          <PhotoPlaceholder label="Foto do ambiente interno da Regusto" />
        </Reveal>
      </Container>
    </Section>
  );
}
```

- [ ] **Step 2: Verify manually**

Temporarily render `<About />` under the Hero on the homepage, run `npm run dev`, scroll down, and confirm the block fades up into place as it enters the viewport (the `Reveal` behavior already verified in Task 5). Revert the temporary render.

Run: `npm run build` — expect no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/About.tsx
git commit -m "feat: add About section"
```

---

### Task 10: Como Funciona — editorial scroll experience (the site's one big motion moment)

Per spec §11.2/§11.3: this section presents Self-service, Marmitex and Grelhados as three **parallel formats**, not three generic feature cards and not a numbered 3-step process. Structural inspiration only from the 21st.dev "Horizontal Feature Reveal" reference (**not installed, not copied**) — a pinned horizontal scroll-jack that this task rebuilds from scratch with Regusto's own tokens, copy, and photography, fixing two things the reference got wrong for this project: it treated "01/02/03" as literal step numbers, and it left the horizontal scroll active under `prefers-reduced-motion`. Neither carries over here. This is the **only other section besides the Hero allowed to use scroll-scrub** — every other section stays on the plain one-shot `Reveal` fade-up.

**Files:**
- Create: `src/components/HowItWorks.tsx`

**Interfaces:**
- Consumes: `gsap`, `SplitText`, `useGSAP` (`@/lib/gsap`), `Reveal`, `PhotoPlaceholder` (`@/components/ui/*`)
- Produces: `HowItWorks()` with `id="como-funciona"`

- [ ] **Step 1: Write the component**

```tsx
// src/components/HowItWorks.tsx
"use client";

import { useRef } from "react";
import { gsap, SplitText, useGSAP } from "@/lib/gsap";
import { Reveal } from "@/components/ui/Reveal";
import { PhotoPlaceholder } from "@/components/ui/PhotoPlaceholder";

interface ServiceFormat {
  name: string;
  description: string;
  photoLabel: string;
}

// Three parallel ways to eat at Regusto, not a sequence — no 01/02/03 numbering
// (spec §11.2). The format name itself is the giant typographic element.
const FORMATS: ServiceFormat[] = [
  {
    name: "Self-service",
    description:
      "Sirva-se à vontade no buffet por quilo, com grelhados, saladas e acompanhamentos renovados todos os dias.",
    photoLabel: "Balcão do self-service da Regusto com pratos montados",
  },
  {
    name: "Marmitex",
    description:
      "Praticidade para o dia a dia: marmitex montado na hora, pronto pra retirar ou pedir pelo WhatsApp.",
    photoLabel: "Marmitex fechado, pronto para viagem",
  },
  {
    name: "Grelhados",
    description: "Peça um grelhado à la carte direto no balcão e leve pra casa na hora que preferir.",
    photoLabel: "Grelhado servido em prato, close no detalhe da carne",
  },
];

export function HowItWorks() {
  const sectionRef = useRef<HTMLElement>(null);
  const wrapperRef = useRef<HTMLDivElement>(null);
  const trackRef = useRef<HTMLDivElement>(null);

  useGSAP(
    () => {
      const mm = gsap.matchMedia();

      // 1024px mirrors the `lg` breakpoint used in the JSX below — if that
      // breakpoint is ever customized in the Tailwind theme, update this too.
      // Below 1024px, OR under prefers-reduced-motion at any width, this branch
      // never runs and the plain stacked markup (already reduced-motion-safe via
      // Reveal) is what renders — no motionless version of the pinned layout.
      mm.add("(min-width: 1024px) and (prefers-reduced-motion: no-preference)", () => {
        if (!wrapperRef.current || !trackRef.current) return;

        const panels = gsap.utils.toArray<HTMLElement>(".format-panel", trackRef.current);
        const slice = 100 / panels.length;
        const splits: SplitText[] = [];

        gsap.to(trackRef.current, {
          xPercent: (-100 * (panels.length - 1)) / panels.length,
          ease: "none",
          scrollTrigger: {
            trigger: wrapperRef.current,
            start: "top top",
            end: "bottom bottom",
            scrub: true,
          },
        });

        panels.forEach((panel, i) => {
          const start = `${i * slice}% top`;
          const end = `${(i + 1) * slice}% top`;

          const name = panel.querySelector<HTMLElement>(".format-name");
          if (name) {
            const split = new SplitText(name, { type: "lines", mask: "lines" });
            splits.push(split);
            gsap.from(split.lines, {
              yPercent: 100,
              duration: 0.8,
              ease: "power3.out",
              scrollTrigger: { trigger: wrapperRef.current, start, end, toggleActions: "play none none reverse" },
            });
          }

          const image = panel.querySelector<HTMLElement>(".format-image");
          if (image) {
            gsap.fromTo(
              image,
              { xPercent: 8 },
              { xPercent: -8, ease: "none", scrollTrigger: { trigger: wrapperRef.current, start, end, scrub: true } }
            );
          }
        });

        return () => splits.forEach((split) => split.revert());
      });

      return () => mm.revert();
    },
    { scope: sectionRef }
  );

  return (
    <section id="como-funciona" ref={sectionRef} className="relative bg-secondary text-neutral-100">
      {/* Desktop, motion-safe: pinned horizontal track — the site's one big
          scroll moment (spec §11.3). Hidden by CSS, not JS, below 1024px or
          under reduced motion, so the stacked fallback below is what's live. */}
      <div ref={wrapperRef} className="hidden lg:motion-safe:block lg:motion-safe:h-[300vh]">
        <div className="sticky top-0 flex h-screen overflow-hidden">
          <div ref={trackRef} className="flex">
            {FORMATS.map((format) => (
              <div key={format.name} className="format-panel flex h-screen w-screen shrink-0 items-center gap-16 px-16">
                <div className="w-1/2">
                  <h3 className="format-name font-heading text-[clamp(2.75rem,6vw,5.5rem)] font-medium leading-[1.02]">
                    {format.name}
                  </h3>
                  <p className="mt-6 max-w-md text-lg text-neutral-200">{format.description}</p>
                </div>
                <div className="format-image h-[70vh] w-1/2 overflow-hidden">
                  <PhotoPlaceholder
                    label={format.photoLabel}
                    aspectClassName="h-full"
                    roundedClassName=""
                    className="h-full w-full"
                  />
                </div>
              </div>
            ))}
          </div>
        </div>
      </div>

      {/* Mobile/tablet AND reduced-motion fallback — same stacked pattern used
          everywhere else on the site (Reveal), no bespoke mobile-only mechanic. */}
      <div className="block px-6 py-24 sm:px-10 lg:motion-safe:hidden">
        <div className="mx-auto flex max-w-xl flex-col gap-16">
          {FORMATS.map((format, i) => (
            <Reveal key={format.name} delay={i * 0.1}>
              <h3 className="font-heading text-[clamp(2.75rem,6vw,5.5rem)] font-medium leading-[1.02]">
                {format.name}
              </h3>
              <p className="mt-4 text-neutral-200">{format.description}</p>
              <PhotoPlaceholder label={format.photoLabel} roundedClassName="" className="mt-6" />
            </Reveal>
          ))}
        </div>
      </div>
    </section>
  );
}
```

- [ ] **Step 2: Verify manually**

Temporarily render `<HowItWorks />` under the Hero on the homepage, run `npm run dev`.
- At ≥1024px width with no reduced-motion preference: scroll through the section and confirm it pins, the three panels (Self-service, Marmitex, Grelhados) track horizontally with the scroll, each format name reveals via a line-mask as its panel comes into focus, no numeral (01/02/03 or similar) is rendered anywhere, and normal vertical scrolling resumes cleanly right after the third panel.
- While scrolled into the pinned section, use Page Down/Space/arrow keys: confirm the page keeps scrolling normally (the pin must never trap keyboard scroll).
- Resize below 1024px (or use devtools' device toolbar): confirm the pinned/horizontal markup disappears entirely and the three formats render as a plain vertical stack, each fading up via `Reveal`.
- Enable "reduce motion" (OS setting or devtools rendering emulation) at a **desktop** width (≥1024px): confirm the section renders the same static stacked layout as mobile — never a motionless version of the horizontal layout.
- Inspect the DOM (or use a screen reader) to confirm each panel's name and description are ordered before its photo, regardless of the left/right visual arrangement.

Run: `npm run build` — expect no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/HowItWorks.tsx
git commit -m "feat: add Como Funciona editorial scroll experience for service formats"
```

---

### Task 11: Cardápio (menu) section

**Files:**
- Create: `src/components/Menu.tsx`

**Interfaces:**
- Consumes: `gsap`, `ScrollTrigger`, `useGSAP` (`@/lib/gsap`), `menuCategories`, `formatPriceBRL` (`@/content/menu`), `siteContent`, `buildWhatsAppOrderUrl`, `Section`, `Container`, `SectionHeading`, `Button`
- Produces: `Menu()` with `id="cardapio"` — note this is the page's menu SECTION, distinct from the shared `Menu`/`Close` icon components from Task 4 (different files, no name collision since icons are imported as `MenuIcon`)

- [ ] **Step 1: Write the component**

```tsx
// src/components/Menu.tsx
"use client";

import { useRef } from "react";
import { gsap, ScrollTrigger, useGSAP } from "@/lib/gsap";
import { Section } from "@/components/ui/Section";
import { Container } from "@/components/ui/Container";
import { SectionHeading } from "@/components/ui/SectionHeading";
import { Button } from "@/components/ui/Button";
import { menuCategories, formatPriceBRL } from "@/content/menu";
import { siteContent } from "@/content/site";
import { buildWhatsAppOrderUrl } from "@/lib/whatsapp";

export function Menu() {
  const containerRef = useRef<HTMLDivElement>(null);
  const dailyDishUrl = buildWhatsAppOrderUrl(
    siteContent.whatsappNumber,
    "Olá! Qual é o prato do dia de hoje na Regusto?"
  );

  useGSAP(
    () => {
      const mm = gsap.matchMedia();

      mm.add({ motionReduced: "(prefers-reduced-motion: reduce)" }, (context) => {
        const { motionReduced } = context.conditions as { motionReduced: boolean };

        if (motionReduced) {
          gsap.set(".menu-category", { autoAlpha: 1, y: 0 });
          return;
        }

        gsap.set(".menu-category", { autoAlpha: 0, y: 24 });

        ScrollTrigger.batch(".menu-category", {
          start: "top 85%",
          onEnter: (batch) =>
            gsap.to(batch, { autoAlpha: 1, y: 0, duration: 0.5, ease: "power3.out", stagger: 0.1 }),
        });
      });

      return () => mm.revert();
    },
    { scope: containerRef }
  );

  return (
    <Section id="cardapio">
      <Container>
        <SectionHeading overline="Cardápio" title="O que você encontra na Regusto" align="center" />

        <div className="mt-8 rounded-md border border-neutral-300 bg-surface-card p-4 text-center text-sm text-neutral-600">
          O prato do dia muda diariamente — confira no{" "}
          <a
            href={siteContent.instagramUrl}
            target="_blank"
            rel="noopener noreferrer"
            className="font-semibold text-primary underline"
          >
            Instagram
          </a>{" "}
          ou{" "}
          <a href={dailyDishUrl} target="_blank" rel="noopener noreferrer" className="font-semibold text-primary underline">
            pergunte no WhatsApp
          </a>
          .
        </div>

        <div ref={containerRef} className="mt-12 grid gap-10 md:grid-cols-2">
          {menuCategories.map((category) => (
            <div key={category.id} className="menu-category">
              <h3 className="font-heading text-xl font-semibold text-neutral-700">{category.name}</h3>
              <ul className="mt-4 divide-y divide-neutral-300">
                {category.items.map((item) => (
                  <li key={item.name} className="flex items-baseline justify-between gap-4 py-3">
                    <div>
                      <p className="font-medium text-neutral-700">
                        {item.name}
                        {item.isPlaceholder && (
                          <span className="ml-2 text-xs font-normal text-neutral-400">(preço a confirmar)</span>
                        )}
                      </p>
                      {item.description && <p className="text-sm text-neutral-500">{item.description}</p>}
                    </div>
                    <span className="shrink-0 font-sans tabular-nums text-neutral-700">
                      {formatPriceBRL(item.price)}
                    </span>
                  </li>
                ))}
              </ul>
            </div>
          ))}
        </div>

        <div className="mt-12 text-center">
          <Button
            href={buildWhatsAppOrderUrl(siteContent.whatsappNumber, siteContent.whatsappDefaultMessage)}
            target="_blank"
            rel="noopener noreferrer"
          >
            Fazer pedido
          </Button>
        </div>
      </Container>
    </Section>
  );
}
```

- [ ] **Step 2: Verify manually**

Temporarily render `<Menu />` on the homepage, run `npm run dev`. Confirm: all 5 categories render with items, prices are right-aligned with the "(preço a confirmar)" tag visible, prices use comma-decimal BRL formatting (e.g. "R$ 42,90"), both the Instagram link and the "pergunte no WhatsApp" link open correctly, and categories fade up in a staggered batch as you scroll to them. Revert the temporary render.

Run: `npm run build` — expect no errors.

- [ ] **Step 3: Commit**

```bash
git add src/components/Menu.tsx
git commit -m "feat: add cardapio section with category batch reveal"
```

---

### Task 12: Gallery (Ambiente) and Practical Info sections

**Files:**
- Create: `src/components/Gallery.tsx`
- Create: `src/components/PracticalInfo.tsx`

**Interfaces:**
- Consumes: `gsap`, `ScrollTrigger`, `useGSAP` (`@/lib/gsap`), `Section`, `Container`, `SectionHeading`, `PhotoPlaceholder`, `Reveal`, `MapPinIcon`, `ClockIcon`, `siteContent`
- Produces: `Gallery()` with `id="ambiente"`, `PracticalInfo()` with `id="informacoes"`

- [ ] **Step 1: `src/components/Gallery.tsx`**

```tsx
"use client";

import { useRef } from "react";
import { gsap, ScrollTrigger, useGSAP } from "@/lib/gsap";
import { Section } from "@/components/ui/Section";
import { Container } from "@/components/ui/Container";
import { SectionHeading } from "@/components/ui/SectionHeading";
import { PhotoPlaceholder } from "@/components/ui/PhotoPlaceholder";

// PLACEHOLDER: rótulos descritivos até receber as fotos reais (spec §6).
const GALLERY_ITEMS = [
  "Prato de grelhados montado",
  "Balcão do self-service",
  "Marmitex pronto para viagem",
  "Salão de refeições",
  "Detalhe de um prato finalizado",
  "Ambiente externo da Regusto",
];

export function Gallery() {
  const containerRef = useRef<HTMLDivElement>(null);

  useGSAP(
    () => {
      const mm = gsap.matchMedia();

      mm.add({ motionReduced: "(prefers-reduced-motion: reduce)" }, (context) => {
        const { motionReduced } = context.conditions as { motionReduced: boolean };

        if (motionReduced) {
          gsap.set(".gallery-item", { clipPath: "inset(0% 0% 0% 0%)", autoAlpha: 1 });
          return;
        }

        gsap.set(".gallery-item", { clipPath: "inset(15% 0% 0% 0%)", autoAlpha: 0 });

        ScrollTrigger.batch(".gallery-item", {
          start: "top 90%",
          onEnter: (batch) =>
            gsap.to(batch, {
              clipPath: "inset(0% 0% 0% 0%)",
              autoAlpha: 1,
              duration: 0.7,
              ease: "power2.out",
              stagger: 0.1,
            }),
        });
      });

      return () => mm.revert();
    },
    { scope: containerRef }
  );

  return (
    <Section id="ambiente" className="bg-neutral-200">
      <Container>
        <SectionHeading overline="Ambiente" title="Um pouco do dia a dia da Regusto" align="center" />
        <div ref={containerRef} className="mt-12 grid gap-4 sm:grid-cols-2 lg:grid-cols-3">
          {GALLERY_ITEMS.map((label) => (
            <div key={label} className="gallery-item transition-transform duration-300 hover:scale-[1.03]">
              <PhotoPlaceholder label={label} />
            </div>
          ))}
        </div>
      </Container>
    </Section>
  );
}
```

- [ ] **Step 2: `src/components/PracticalInfo.tsx`**

```tsx
import { Section } from "@/components/ui/Section";
import { Container } from "@/components/ui/Container";
import { SectionHeading } from "@/components/ui/SectionHeading";
import { Reveal } from "@/components/ui/Reveal";
import { MapPinIcon, ClockIcon } from "@/components/ui/icons";
import { siteContent } from "@/content/site";

export function PracticalInfo() {
  const { address, businessHours, paymentMethods } = siteContent;

  return (
    <Section id="informacoes" className="bg-secondary text-neutral-100">
      <Container className="grid gap-10 md:grid-cols-2">
        <Reveal>
          <SectionHeading overline="Informações práticas" title="Onde e quando" />

          <div className="mt-6 flex items-start gap-3">
            <MapPinIcon className="mt-1 h-5 w-5 shrink-0 text-accent" />
            <p>
              {address.street}, {address.neighborhood}
              <br />
              {address.city} - {address.state}, {address.zip}
            </p>
          </div>

          <div className="mt-4 flex items-start gap-3">
            <ClockIcon className="mt-1 h-5 w-5 shrink-0 text-accent" />
            <p>{businessHours.display}</p>
          </div>

          <p className="mt-6 text-sm text-neutral-300">Formas de pagamento: {paymentMethods.join(", ")}.</p>
        </Reveal>

        <Reveal delay={0.1}>
          <iframe
            src={siteContent.mapsEmbedUrl}
            className="h-72 w-full rounded-lg border-0 md:h-full"
            loading="lazy"
            referrerPolicy="no-referrer-when-downgrade"
            title={`Mapa de localização da ${siteContent.brandName}`}
          />
        </Reveal>
      </Container>
    </Section>
  );
}
```

- [ ] **Step 3: Verify manually**

Temporarily render both on the homepage, run `npm run dev`. Confirm: gallery photo placeholders clip-reveal on scroll and zoom slightly on hover; the practical info section shows the correct address, the exact `businessHours.display` string (not a re-typed value), payment methods, and an embedded, correctly-centered Google Map. Revert the temporary render.

Run: `npm run build` — expect no errors.

- [ ] **Step 4: Commit**

```bash
git add src/components/Gallery.tsx src/components/PracticalInfo.tsx
git commit -m "feat: add gallery and practical info sections"
```

---

### Task 13: Footer

**Files:**
- Create: `src/components/Footer.tsx`

**Interfaces:**
- Consumes: `Container` (`@/components/ui/Container`), `WhatsAppOrderButton`, `siteContent`
- Produces: `Footer()` with `id="contato"`

- [ ] **Step 1: Write the component**

```tsx
import { Container } from "@/components/ui/Container";
import { WhatsAppOrderButton } from "@/components/WhatsAppOrderButton";
import { siteContent } from "@/content/site";

export function Footer() {
  return (
    <footer id="contato" className="bg-neutral-700 py-16 text-neutral-200">
      <Container className="flex flex-col items-start gap-6 md:flex-row md:items-center md:justify-between">
        <div>
          <p className="font-heading text-2xl text-neutral-100">{siteContent.brandName}</p>
          <p className="mt-2 text-sm">
            {siteContent.address.street}, {siteContent.address.neighborhood} — {siteContent.address.city}/
            {siteContent.address.state}
          </p>
          <p className="text-sm">{siteContent.phone}</p>
          <p className="mt-4 text-xs text-neutral-400">CNPJ {siteContent.cnpj}</p>
        </div>

        <WhatsAppOrderButton />
      </Container>
    </footer>
  );
}
```

- [ ] **Step 2: Verify and commit**

Run: `npm run build` — expect no errors.

```bash
git add src/components/Footer.tsx
git commit -m "feat: add footer with contact details and order CTA"
```

---

### Task 14: Assemble the homepage

**Files:**
- Modify: `src/app/page.tsx`

**Interfaces:**
- Consumes: `Header`, `Hero`, `About`, `HowItWorks`, `Menu`, `Gallery`, `PracticalInfo`, `Footer` (all components from Tasks 7–13)
- Produces: the complete, navigable homepage at `/`

- [ ] **Step 1: Replace the default homepage**

```tsx
// src/app/page.tsx
import { Header } from "@/components/Header";
import { Hero } from "@/components/Hero";
import { About } from "@/components/About";
import { HowItWorks } from "@/components/HowItWorks";
import { Menu } from "@/components/Menu";
import { Gallery } from "@/components/Gallery";
import { PracticalInfo } from "@/components/PracticalInfo";
import { Footer } from "@/components/Footer";

export default function Home() {
  return (
    <>
      <Header />
      <main>
        <Hero />
        <About />
        <HowItWorks />
        <Menu />
        <Gallery />
        <PracticalInfo />
      </main>
      <Footer />
    </>
  );
}
```

- [ ] **Step 2: Verify the full page end-to-end**

Run: `npm run dev`, open `http://localhost:3000`. Confirm: every section renders in order (Hero → Sobre → Como Funciona → Cardápio → Ambiente → Informações Práticas → Footer), every header nav link scrolls to the matching section id, and there are no console errors.

Run: `npm run build` — expect no errors.

- [ ] **Step 3: Commit**

```bash
git add src/app/page.tsx
git commit -m "feat: assemble homepage from all sections"
```

---

### Task 15: Final responsive, accessibility, and reduced-motion QA

**Files:**
- Modify: any file where an issue is found during this pass (no new files expected)

**Interfaces:**
- Consumes: the complete site from Task 14
- Produces: a verified, shippable MVP

- [ ] **Step 1: Responsive pass**

Run `npm run dev`. Using browser devtools device toolbar, check the full page at 375px (mobile), 768px (tablet), and 1440px (desktop) widths. Confirm: no horizontal scrollbar at any width, the mobile nav overlay works, the menu/gallery grids reflow to single/double column on narrow widths, and text never overflows its container.

- [ ] **Step 2: Reduced-motion pass**

Enable "prefers-reduced-motion: reduce" (devtools rendering tab, or OS setting), reload the full page, and scroll through it. Confirm: Hero photo, headline, and signature icon all appear instantly at final state (no fade/scale/line-mask/draw); Como Funciona renders as the static stacked layout at every width, including desktop (never a frozen horizontal layout); every other `Reveal`-wrapped section, the menu batch reveal, and the gallery clip reveal all show their final state immediately with no animation.

- [ ] **Step 3: Keyboard and screen-reader-basics pass**

Tab through the page from the top. Confirm: focus reaches the header nav links, the mobile menu toggle, every WhatsApp CTA, and the footer — in a sensible order — and focus is visually indicated. Confirm every `PhotoPlaceholder` and icon has appropriate `alt`/`aria-label`/`aria-hidden` (already set in the components; verify nothing was dropped during integration).

- [ ] **Step 4: Fix anything found**

If any check in Steps 1–3 fails, fix it in the relevant component file now, re-run the specific check, and confirm it passes.

- [ ] **Step 5: Final build check and commit**

Run: `npm run build` — expect no errors.

```bash
git add -A
git commit -m "chore: final responsive, accessibility, and reduced-motion QA pass"
```

- [ ] **Step 6: Flag remaining client-dependent follow-ups (not part of this plan's scope)**

Confirm these are still tracked in the spec (§6, §2.4) and communicate them to the client before launch — they are content/business decisions, not engineering tasks: real menu items and prices, real photography (hero, about, gallery), real "Sobre" text, confirmation of the business-hours discrepancy (then flip `siteContent.businessHours.confirmedForLaunch` to `true`), and confirmation of whether "Paulo Celso Alarcon" should appear as an owner/chef credit.

---

## Self-Review Notes

- **Spec coverage:** §1 (context) → Task 1; §2.4/§6 hours rule → Task 3 Step 1 + Task 12 Step 2 (renders `businessHours.display` directly, never re-typed); §3 (Approach A, WhatsApp) → Tasks 3, 6, 11, 13; §4 (Design DNA) → Tasks 2, 5, 8, 10; §5 (IA) → Tasks 7–13 (anchors match section ids exactly); §6 (content model, placeholders) → Tasks 3, 9, 10, 11, 12; §7 (a11y/perf) → Reveal/Hero/HowItWorks/Menu/Gallery `matchMedia` guards + Task 15; §8 (testing = manual, no unit tests) → every task's verify step; §9 (out of scope) → no CMS/form/i18n/`\`/cardapio\``/WebGL anywhere in the plan (WebGL/3D exception is judgment-only, per §11.6, never scheduled as a task); §11 (21st.dev research, motion hierarchy, Hero photo layer, Como Funciona editorial scroll experience) → Tasks 5, 8, 10 — no 21st component installed or copied verbatim anywhere in this plan.
- **Placeholder scan:** no "TBD"/"TODO"/"implement later" in any step; every code block is complete and runnable as written; content placeholders are real, working code marked with explicit `isPlaceholder`/`PhotoPlaceholder`/`{/* PLACEHOLDER */}` conventions, not vague instructions.
- **Type consistency checked:** `SiteContent`/`BusinessHours`/`SiteAddress` (Task 3) match every consumer's destructuring in Tasks 6, 7, 8, 11, 12, 13. `MenuCategory`/`MenuItem`/`formatPriceBRL` (Task 3) match Task 11's usage exactly. `ButtonProps` (Task 4) supports the `href`+`target`+`rel` combination used by Tasks 6 and 11. `PhotoPlaceholder`'s new `roundedClassName` prop (Task 4) is used consistently by Tasks 8, 10, and 12 wherever a full-bleed/square-cornered treatment is needed. Section `id`s (`hero`, `sobre`, `como-funciona`, `cardapio`, `ambiente`, `informacoes`, `contato`) are consistent between Header's `NAV_ITEMS` (Task 7) and the components that define them (Tasks 8–13). `gsap`/`ScrollTrigger`/`SplitText`/`useGSAP` are imported from `@/lib/gsap` everywhere after Task 5, never re-imported/re-registered from the raw packages, and never mixed with Framer Motion.
- **Motion hierarchy checked (spec §11.3):** only Task 8 (Hero) and Task 10 (Como Funciona) use scroll-scrub; Tasks 9, 11, 12, 13 (About, Menu, Gallery/PracticalInfo, Footer) use only the one-shot `Reveal`/batch-reveal pattern already established in Task 5 — no task adds a second competing "big moment."
