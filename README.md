# Tehami's Blog

Personal blog built with [Hugo](https://gohugo.io/) and the [PaperMod](https://github.com/adityatelange/hugo-PaperMod) theme.  
Includes sections for **Tech** and **Books**, plus static pages like **About** and **Contact**.

---

## 🚀 Setup

### 1. Install Hugo (Extended Version Required)
Check your version:
```
hugo version
```

### 2. Clone Repo
```
git clone <your-repo-url> tehamis-blog
cd blog
```

### 3. Install Theme
```
git submodule add https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
git submodule update --remote --merge
```

### 4. Run in Development
```
hugo server -D
```

## Build for Production
```
hugo
```

## Content Structure
```
content/
 ├── tech/        # Technical posts
 ├── books/       # Book reviews
 ├── about.md     # About page
 └── contact.md   # Contact page
```

