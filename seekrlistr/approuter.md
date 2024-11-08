# Project Directory Structure

Below is the directory structure for the **Seekr and Listr** platform using Next.js App Router.

```plaintext
seekr-listr/
├── README.md
├── package.json
├── package-lock.json
├── .gitignore
├── .editorconfig
├── LICENSE
├── frontend/
│   ├── package.json
│   ├── tsconfig.json
│   ├── next.config.js 
│   ├── public/
│   │   ├── favicon.ico
│   │   └── images/
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── page.tsx
│   │   ├── seekr/
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   └── ...
│   │   ├── listr/
│   │   │   ├── dashboard/
│   │   │   │   └── page.tsx
│   │   │   └── ...
│   │   ├── unauthorized/
│   │   │   └── page.tsx
│   │   ├── api/
│   │   │   ├── auth/
│   │   │   │   └── route.ts
│   │   │   └── ...
│   │   ├── components/
│   │   │   ├── Sidebar/
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   └── RoleToggle.tsx
│   │   │   ├── Navigation/
│   │   │   │   ├── SeekrNavigation.tsx
│   │   │   │   ├── ListrNavigation.tsx
│   │   │   │   └── Navigation.tsx
│   │   │   ├── ProtectedRoute.tsx (if needed)
│   │   │   └── ...
│   │   ├── contexts/
│   │   │   ├── AuthContext.tsx
│   │   │   └── RoleContext.tsx
│   │   ├── hooks/
│   │   │   └── useAuth.ts
│   │   ├── utils/
│   │   │   └── auth.ts
│   │   ├── styles/
│   │   │   └── globals.css
│   │   └── middleware.ts
│   ├── config/
│   ├── tests/
│   └── .env
├── backend/
│   ├── ... (same as before)
├── ai-ml/
│   ├── ...
├── database/
│   ├── ...
├── docs/
│   ├── ...
├── scripts/
├── config/
├── tests/
├── logs/
├── docker/
├── .github/
└── .vscode/