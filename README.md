# Personal Injury Case Value Estimator

A premium, modern web application built with Vue 3 to estimate the potential value of personal injury cases. The UI follows a "Dark Mode Premium" aesthetic, focused on clean typography, smooth interactions, and a professional look and feel.

## 🌟 Features

- **Economic Loss Calculation**: Easily input medical expenses, lost income, property damage, and out-of-pocket costs.
- **Pain & Suffering Multiplier**: Adjustable slider (1x - 5x) to account for non-economic damages based on injury severity.
- **Comparative Fault Deduction**: Implements modified comparative fault rules (e.g., Florida's 50% rule where compensation drops to $0 if the plaintiff is $\ge$ 50% at fault).
- **Insurance Policy Limits**: Optional cap to adjust the final estimated settlement based on available insurance coverage.
- **Contextual Tooltips**: Interactive "help" popups on every input field providing real-world examples and definitions for legal terms.
- **Dynamic Visual Breakdown**: Real-time percentage visualization showing the composition of the estimated value (Economic vs. Non-Economic).
- **Fully Responsive**: Optimized for desktop, tablet, and mobile viewing.

## 🛠️ Tech Stack

- **Framework**: [Vue 3](https://vuejs.org/) (Composition API)
- **Tooling**: [Vite](https://vitejs.dev/)
- **Icons**: [Lucide for Vue](https://lucide.dev/guide/packages/lucide-vue-next)
- **Styling**: Vanilla CSS (Modular CSS with custom CSS properties for theming)

## 🚀 Getting Started

### Prerequisites
Make sure you have Node.js installed on your machine (`>= 20.19.0` recommended).

### Installation

1. Clone the repository:
```bash
git clone https://github.com/germardones/calculator_Injury.git
cd calculator_Injury
```

2. Install dependencies:
```bash
npm install
```

3. Start the development server:
```bash
npm run dev
```

4. Build for production:
```bash
npm run build
```

## ⚖️ Legal Disclaimer
*This tool provides a generalized estimate based on standard insurance formulas. It does not constitute legal advice. Every accident case is unique, and actual case values depend on numerous factors including jurisdiction, evidence, and specific case details.*
