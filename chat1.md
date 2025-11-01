Perfetto! Ti spiego passo passo come implementare correttamente la **versione 3 con colori e informazioni avanzate** per bash:

## **Implementazione completa della versione 3**

### **Passo 1: Apri il file di configurazione**
```bash
nano ~/.bashrc
# oppure con il tuo editor preferito
vim ~/.bashrc
```

### **Passo 2: Aggiungi la funzione git_info**
Aggiungi questa funzione alla fine del file `~/.bashrc`:

```bash
git_info() {
    # Controlla se siamo in un repository Git
    local branch=$(git symbolic-ref HEAD 2>/dev/null | cut -d/ -f3-)
    
    if [[ -n $branch ]]; then
        local status=""
        local color="01;32"  # Verde di default
        
        # Controlla se ci sono modifiche non committate
        if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
            status="*"
            color="01;33"  # Giallo se ci sono modifiche
        fi
        
        # Controlla se ci sono modifiche staged
        if [[ -n $(git diff --cached --name-only 2>/dev/null) ]]; then
            status="${status}+"
        fi
        
        # Controlla se siamo ahead/behind del remote
        local ahead_behind=$(git status --porcelain=v1 --branch 2>/dev/null | head -n1)
        if [[ $ahead_behind =~ \[ahead\ ([0-9]+)\] ]]; then
            status="${status}↑"
        fi
        if [[ $ahead_behind =~ \[behind\ ([0-9]+)\] ]]; then
            status="${status}↓"
        fi
        
        echo " (\\[\\033[${color}m\\]$branch$status\\[\\033[00m\\])"
    fi
}
```

### **Passo 3: Imposta il prompt**
Aggiungi questa riga dopo la funzione:

```bash
export PS1='\\[\\033[01;32m\\]\\u@\\h\\[\\033[00m\\]:\\[\\033[01;34m\\]\\w\\$(git_info)\\[\\033[00m\\]\\$ '
```

### **Passo 4: Spiegazione dei codici colore**
```bash
# \\[\\033[01;32m\\] = Verde grassetto
# \\[\\033[01;34m\\] = Blu grassetto  
# \\[\\033[01;33m\\] = Giallo grassetto (per modifiche)
# \\[\\033[00m\\]   = Reset colore
```

### **Passo 5: Versione migliorata (opzionale)**
Se vuoi una versione ancora più completa, sostituisci la funzione con questa:

```bash
git_info() {
    # Verifica se siamo in un repository Git
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        return 0
    fi
    
    local branch=$(git symbolic-ref --short HEAD 2>/dev/null || git describe --tags --always 2>/dev/null)
    local status=""
    local color="01;32"  # Verde di default
    
    # Controlla lo stato del working directory
    if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
        local modified=$(git status --porcelain 2>/dev/null | grep "^.M" | wc -l)
        local added=$(git status --porcelain 2>/dev/null | grep "^A." | wc -l)
        local deleted=$(git status --porcelain 2>/dev/null | grep "^.D" | wc -l)
        local untracked=$(git status --porcelain 2>/dev/null | grep "^??" | wc -l)
        
        if [[ $modified -gt 0 ]]; then status="${status}M$modified"; fi
        if [[ $added -gt 0 ]]; then status="${status}A$added"; fi
        if [[ $deleted -gt 0 ]]; then status="${status}D$deleted"; fi
        if [[ $untracked -gt 0 ]]; then status="${status}?$untracked"; fi
        
        color="01;33"  # Giallo se ci sono modifiche
    fi
    
    # Controlla se ci sono file staged
    if [[ -n $(git diff --cached --name-only 2>/dev/null) ]]; then
        status="${status}+"
        color="01;36"  # Ciano se ci sono file staged
    fi
    
    # Controlla ahead/behind
    local upstream=$(git rev-parse --abbrev-ref @{upstream} 2>/dev/null)
    if [[ -n $upstream ]]; then
        local ahead=$(git rev-list --count HEAD..@{upstream} 2>/dev/null)
        local behind=$(git rev-list --count @{upstream}..HEAD 2>/dev/null)
        
        if [[ $ahead -gt 0 ]]; then status="${status}↓$ahead"; fi
        if [[ $behind -gt 0 ]]; then status="${status}↑$behind"; fi
    fi
    
    if [[ -n $status ]]; then
        echo " (\\[\\033[${color}m\\]$branch\\[\\033[01;31m\\]$status\\[\\033[00m\\])"
    else
        echo " (\\[\\033[${color}m\\]$branch\\[\\033[00m\\])"
    fi
}
```

### **Passo 6: Applica le modifiche**
```bash
source ~/.bashrc
```

### **Passo 7: Test**
Vai in un repository Git e verifica il risultato:
```bash
cd /path/to/your/git/repo
# Il prompt dovrebbe mostrare qualcosa come:
# user@hostname:~/my-repo (main) $ 
# oppure con modifiche:
# user@hostname:~/my-repo (main*) $ 
```

## **Significato dei simboli**
- `*` = Ci sono modifiche non committate
- `+` = Ci sono modifiche staged
- `↑3` = Sei 3 commit avanti rispetto al remote
- `↓2` = Sei 2 commit indietro rispetto al remote
- `M2` = 2 file modificati
- `A1` = 1 file aggiunto
- `D1` = 1 file eliminato
- `?3` = 3 file non tracciati

## **Colori**
- **Verde**: Branch pulito, tutto sincronizzato
- **Giallo**: Ci sono modifiche non committate  
- **Ciano**: Ci sono modifiche staged per il commit

Questa implementazione ti darà un prompt molto informativo che ti dice subito lo stato del tuo repository Git!

Ah, vedo il problema! I caratteri di escape non vengono interpretati correttamente. Questo succede perché ci sono troppi backslash. Quando scrivi direttamente nel `~/.bashrc`, la sintassi è diversa.

## **Correzione del prompt**

Sostituisci la riga del prompt con questa versione corretta:

```bash
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w$(git_info)\[\033[00m\]\$ '
```

## **E correggi anche la funzione git_info:**

```bash
git_info() {
    # Verifica se siamo in un repository Git
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        return 0
    fi
    
    local branch=$(git symbolic-ref --short HEAD 2>/dev/null || git describe --tags --always 2>/dev/null)
    local status=""
    local color="01;32"  # Verde di default
    
    # Controlla lo stato del working directory
    if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
        local modified=$(git status --porcelain 2>/dev/null | grep "^.M" | wc -l)
        local added=$(git status --porcelain 2>/dev/null | grep "^A." | wc -l)
        local deleted=$(git status --porcelain 2>/dev/null | grep "^.D" | wc -l)
        local untracked=$(git status --porcelain 2>/dev/null | grep "^??" | wc -l)
        
        if [[ $modified -gt 0 ]]; then status="${status}M$modified"; fi
        if [[ $added -gt 0 ]]; then status="${status}A$added"; fi
        if [[ $deleted -gt 0 ]]; then status="${status}D$deleted"; fi
        if [[ $untracked -gt 0 ]]; then status="${status}?$untracked"; fi
        
        color="01;33"  # Giallo se ci sono modifiche
    fi
    
    # Controlla se ci sono file staged
    if [[ -n $(git diff --cached --name-only 2>/dev/null) ]]; then
        status="${status}+"
        color="01;36"  # Ciano se ci sono file staged
    fi
    
    # Controlla ahead/behind
    local upstream=$(git rev-parse --abbrev-ref @{upstream} 2>/dev/null)
    if [[ -n $upstream ]]; then
        local ahead=$(git rev-list --count HEAD..@{upstream} 2>/dev/null)
        local behind=$(git rev-list --count @{upstream}..HEAD 2>/dev/null)
        
        if [[ $ahead -gt 0 ]]; then status="${status}↓$ahead"; fi
        if [[ $behind -gt 0 ]]; then status="${status}↑$behind"; fi
    fi
    
    if [[ -n $status ]]; then
        echo " (\[\033[${color}m\]$branch\[\033[01;31m\]$status\[\033[00m\])"
    else
        echo " (\[\033[${color}m\]$branch\[\033[00m\])"
    fi
}
```

## **Le differenze chiave:**

1. **Nel PS1**: usa `\[` invece di `\\[`
2. **Nella funzione**: usa `\[` invece di `\\[`
3. **Rimuovi i backslash doppi** che servono solo quando passi attraverso echo

## **Applica le modifiche:**

```bash
source ~/.bashrc
```

## **Risultato atteso:**

Ora dovresti vedere qualcosa come:
```
user@hostname:~/my-repo (main) $ 
```

Con i colori:
- **user@hostname** in verde
- **:/path** in blu  
- **(branch)** in verde (o giallo/ciano se ci sono modifiche)

Se ancora non funziona, puoi anche provare questa versione ancora più semplificata:

```bash
git_info() {
    local branch=$(git branch 2>/dev/null | grep '*' | sed 's/* //')
    if [[ -n $branch ]]; then
        local status=""
        if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
            status="*"
        fi
        echo " ($branch$status)"
    fi
}

export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[01;33m\]$(git_info)\[\033[00m\]\$ '
```

Dimmi se ora funziona correttamente!

Perfetto! Ora vedo il problema: i caratteri di escape per i colori `\[` e `\]` vengono stampati letteralmente invece di essere interpretati. Questo succede perché nella funzione `git_info()` non devi usare `\[` e `\]` - quelli servono solo direttamente nel PS1.

## **Correzione della funzione git_info:**

Sostituisci la funzione `git_info()` nel tuo `~/.bashrc` con questa versione corretta:

```bash
git_info() {
    # Verifica se siamo in un repository Git
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        return 0
    fi
    
    local branch=$(git symbolic-ref --short HEAD 2>/dev/null || git describe --tags --always 2>/dev/null)
    local status=""
    local color="01;32"  # Verde di default
    
    # Controlla lo stato del working directory
    if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
        local modified=$(git status --porcelain 2>/dev/null | grep "^.M" | wc -l)
        local added=$(git status --porcelain 2>/dev/null | grep "^A." | wc -l)
        local deleted=$(git status --porcelain 2>/dev/null | grep "^.D" | wc -l)
        local untracked=$(git status --porcelain 2>/dev/null | grep "^??" | wc -l)
        
        if [[ $modified -gt 0 ]]; then status="${status}M$modified"; fi
        if [[ $added -gt 0 ]]; then status="${status}A$added"; fi
        if [[ $deleted -gt 0 ]]; then status="${status}D$deleted"; fi
        if [[ $untracked -gt 0 ]]; then status="${status}?$untracked"; fi
        
        color="01;33"  # Giallo se ci sono modifiche
    fi
    
    # Controlla se ci sono file staged
    if [[ -n $(git diff --cached --name-only 2>/dev/null) ]]; then
        status="${status}+"
        color="01;36"  # Ciano se ci sono file staged
    fi
    
    # Controlla ahead/behind
    local upstream=$(git rev-parse --abbrev-ref @{upstream} 2>/dev/null)
    if [[ -n $upstream ]]; then
        local ahead=$(git rev-list --count HEAD..@{upstream} 2>/dev/null)
        local behind=$(git rev-list --count @{upstream}..HEAD 2>/dev/null)
        
        if [[ $ahead -gt 0 ]]; then status="${status}↓$ahead"; fi
        if [[ $behind -gt 0 ]]; then status="${status}↑$behind"; fi
    fi
    
    if [[ -n $status ]]; then
        echo " (\033[${color}m$branch\033[01;31m$status\033[00m)"
    else
        echo " (\033[${color}m$branch\033[00m)"
    fi
}
```

## **La differenza chiave:**

**PRIMA (sbagliato):**
```bash
echo " (\\[\\033[${color}m\\]$branch\\[\\033[01;31m\\]$status\\[\\033[00m\\])"
```

**DOPO (corretto):**
```bash
echo " (\033[${color}m$branch\033[01;31m$status\033[00m)"
```

I caratteri `\[` e `\]` servono solo nel PS1 per dire a bash di non contare quei caratteri nella lunghezza del prompt, ma **non** vanno usati nelle funzioni che vengono chiamate dal prompt.

## **Versione semplificata (se preferisci):**

Se vuoi una versione più semplice:

```bash
git_info() {
    local branch=$(git branch 2>/dev/null | grep '*' | sed 's/* //')
    if [[ -n $branch ]]; then
        local status=""
        local color="32"  # Verde
        
        if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
            status="*"
            color="33"  # Giallo
        fi
        
        echo " (\033[${color}m$branch$status\033[0m)"
    fi
}
```

## **Applica le modifiche:**

```bash
source ~/.bashrc
```

## **Risultato atteso:**

Ora dovresti vedere:
```
dpuser@dpserver:/jobs/Git/grocery (master*) $
```

Con il branch colorato correttamente (verde se pulito, giallo se ci sono modifiche).

Prova e dimmi se ora funziona correttamente!

Ah, ora vedo il problema! I codici ANSI `\033` non vengono interpretati. Questo succede perché bash non li sta processando correttamente. Ci sono due soluzioni:

## **Soluzione 1: Funzione che restituisce solo testo (consigliata)**

Modifica la funzione per restituire solo il testo, e metti i colori direttamente nel PS1:

```bash
git_info() {
    # Verifica se siamo in un repository Git
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        return 0
    fi
    
    local branch=$(git symbolic-ref --short HEAD 2>/dev/null || git describe --tags --always 2>/dev/null)
    local status=""
    
    # Controlla lo stato del working directory
    if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
        local modified=$(git status --porcelain 2>/dev/null | grep "^.M" | wc -l)
        local added=$(git status --porcelain 2>/dev/null | grep "^A." | wc -l) 
        local deleted=$(git status --porcelain 2>/dev/null | grep "^.D" | wc -l)
        local untracked=$(git status --porcelain 2>/dev/null | grep "^??" | wc -l)
        
        if [[ $modified -gt 0 ]]; then status="${status}M$modified"; fi
        if [[ $added -gt 0 ]]; then status="${status}A$added"; fi
        if [[ $deleted -gt 0 ]]; then status="${status}D$deleted"; fi
        if [[ $untracked -gt 0 ]]; then status="${status}?$untracked"; fi
    fi
    
    # Controlla se ci sono file staged
    if [[ -n $(git diff --cached --name-only 2>/dev/null) ]]; then
        status="${status}+"
    fi
    
    echo "$branch$status"
}

# Funzione per determinare il colore
git_color() {
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        return 0
    fi
    
    if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
        echo "01;33"  # Giallo se ci sono modifiche
    elif [[ -n $(git diff --cached --name-only 2>/dev/null) ]]; then
        echo "01;36"  # Ciano se ci sono file staged
    else
        echo "01;32"  # Verde se tutto è pulito
    fi
}

# Prompt con colori gestiti direttamente
export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[$(git_color)m\]$(git_info)\[\033[00m\]\$ '
```

**ASPETTA!** Il problema con questa soluzione è che `$(git_color)` non funziona bene nel PS1.

## **Soluzione 2: Usa `\e` invece di `\033` (più semplice)**

```bash
git_info() {
    # Verifica se siamo in un repository Git
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        return 0
    fi
    
    local branch=$(git symbolic-ref --short HEAD 2>/dev/null || git describe --tags --always 2>/dev/null)
    local status=""
    local color="\e[32m"  # Verde di default
    
    # Controlla lo stato del working directory
    if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
        local modified=$(git status --porcelain 2>/dev/null | grep "^.M" | wc -l)
        local added=$(git status --porcelain 2>/dev/null | grep "^A." | wc -l)
        local deleted=$(git status --porcelain 2>/dev/null | grep "^.D" | wc -l) 
        local untracked=$(git status --porcelain 2>/dev/null | grep "^??" | wc -l)
        
        if [[ $modified -gt 0 ]]; then status="${status}M$modified"; fi
        if [[ $added -gt 0 ]]; then status="${status}A$added"; fi
        if [[ $deleted -gt 0 ]]; then status="${status}D$deleted"; fi
        if [[ $untracked -gt 0 ]]; then status="${status}?$untracked"; fi
        
        color="\e[33m"  # Giallo se ci sono modifiche
    fi
    
    # Controlla se ci sono file staged
    if [[ -n $(git diff --cached --name-only 2>/dev/null) ]]; then
        status="${status}+"
        color="\e[36m"  # Ciano se ci sono file staged
    fi
    
    if [[ -n $status ]]; then
        echo " (${color}$branch\e[31m$status\e[0m)"
    else
        echo " (${color}$branch\e[0m)"
    fi
}

export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w$(git_info)\[\033[00m\]\$ '
```

## **Soluzione 3: Versione ultra-semplice (quella che funziona sempre)**

Se le precedenti non funzionano, usa questa versione minimal:

```bash
git_info() {
    local branch=$(git branch 2>/dev/null | grep '*' | sed 's/* //')
    if [[ -n $branch ]]; then
        local status=""
        if [[ -n $(git status --porcelain 2>/dev/null) ]]; then
            status="*"
        fi
        echo " ($branch$status)"
    fi
}

export PS1='\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\w\[\033[01;33m\]$(git_info)\[\033[00m\]\$ '
```

Questa ultima versione mette tutto in giallo quando sei in un repository Git, ma è semplice e **funziona sempre**.

## **Applica una delle soluzioni:**

```bash
source ~/.bashrc
```

Ti consiglio di partire dalla **Soluzione 3** (la più semplice) per verificare che funzioni, poi eventualmente passare alla 2 se vuoi i colori dinamici.

Quale provi per prima?