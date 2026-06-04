
local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/minhdepzai-v/LibraryRobloc/refs/heads/main/RedzLibrary.lua"))()

local Window = redzlib:MakeWindow({
    Title = "Cherry Hub",
    SubTitle = "v3.2 - Final Edition",
    SaveFolder = "CherryMM2"
})

Window:AddMinimizeButton({
    Button = { Image = "rbxassetid://78702423919944", BackgroundTransparency = 0 },
    Corner = { CornerRadius = UDim.new(35, 1) },
})

local lp = game.Players.LocalPlayer
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- CONFIGURAÇÕES GLOBAIS (RESTAURADAS E COMPLETAS)
local ESP_ENABLED      = false
local HITBOX_ENABLED   = false
local KILLAURA_ENABLED = false
local COIN_ENABLED     = false
local FARM_SPEED       = 60
local HITBOX_SIZE      = 10
local KILLAURA_RADIUS  = 10

local selectedPlayer   = nil
local viewEnabled      = false
local flingTargetLoop  = false
local playerEspEnabled = false

local SILENT_AIM_ENABLED = false
local AIMBOT_ENABLED = false
local AUTO_GRAB_GUN = false
_G.AntiFlingEnabled = true

-- NOVO: SMART ANTI-FLING ROBUSTO
task.spawn(function()
    while task.wait() do
        if _G.AntiFlingEnabled and lp.Character and lp.Character:FindFirstChild("Humanoid") then
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and (hrp.Velocity.Magnitude > 50 or hrp.RotVelocity.Magnitude > 50) then
                        for _, part in pairs(p.Character:GetDescendants()) do
                            if part:IsA("BasePart") then part.CanCollide = false end
                        end
                    end
                end
            end
        end
    end
end)



-- SISTEMA DE IDENTIFICAÇÃO DE PAPÉIS (DINÂMICO)
local function getPlayerRole(p)
    if not p or not p.Parent then return "Innocent" end
    local char = p.Character
    local bp = p.Backpack
    -- Verifica faca (Assassino)
    if (char and char:FindFirstChild("Knife")) or (bp and bp:FindFirstChild("Knife")) then
        return "Murderer"
    -- Verifica arma/revolver (Xerife)
    elseif (char and (char:FindFirstChild("Gun") or char:FindFirstChild("Revolver"))) or 
           (bp and (bp:FindFirstChild("Gun") or bp:FindFirstChild("Revolver"))) then
        return "Sheriff"
    end
    return "Innocent"
end

-- SISTEMA DE ESP COM HIGHLIGHT (ESTÁVEL)
local function applyESP(p)
    if p == lp or not p.Character then return end
    
    -- Limpeza de Highlights antigos para evitar lag (Limite 31 do Roblox)
    local old = p.Character:FindFirstChild("CherryHighlight")
    if old then old:Destroy() end

    -- Só aplica se o ESP estiver ligado ou for o player selecionado na Aba Troll
    if not ESP_ENABLED and (not playerEspEnabled or p ~= selectedPlayer) then return end

    local role = getPlayerRole(p)
    
    -- Se for inocente e não for o player selecionado, não mostra nada
    if role == "Innocent" and (not playerEspEnabled or p ~= selectedPlayer) then return end

    local h = Instance.new("Highlight")
    h.Name = "CherryHighlight"
    h.FillTransparency = 0.4
    h.OutlineTransparency = 0
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.Parent = p.Character

    -- Cores REAIS de Identificação
    if role == "Murderer" then 
        h.FillColor = Color3.fromRGB(255, 0, 0) -- Vermelho para Assassino
    elseif role == "Sheriff" then 
        h.FillColor = Color3.fromRGB(0, 120, 255) -- Azul para Xerife
    else 
        h.FillColor = Color3.fromRGB(255, 255, 255) -- Branco para Alvo Troll
    end
end

-- Loop de Atualização do ESP (1 vez por segundo para performance)
task.spawn(function()
    while task.wait(1) do
        for _, p in pairs(Players:GetPlayers()) do
            applyESP(p)
        end
    end
end)






-- FLING NORMAL (REVISADO)
local function executeFling(targetPlayer)
    if not targetPlayer or not targetPlayer.Character then return end
    local myChar = lp.Character
    local myHRP  = myChar and myChar:FindFirstChild("HumanoidRootPart")
    if not myHRP then return end
    local tHRP = targetPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not tHRP then return end

    local savedPos = myHRP.CFrame  
    local bv = Instance.new("BodyVelocity")  
    bv.MaxForce = Vector3.new(1,1,1)*math.huge; bv.Velocity = Vector3.new(9e8, 9e8, 9e8); bv.Parent = myHRP  
    
    local startTime = tick()
    repeat  
        tHRP = targetPlayer.Character and targetPlayer.Character:FindFirstChild("HumanoidRootPart")
        if tHRP then
            -- Penetra e gira no alvo
            myHRP.CFrame = tHRP.CFrame * CFrame.Angles(math.rad(math.random(0,360)), 0, 0)
            myHRP.Velocity = Vector3.new(9e7, 9e8, 9e7)
            myHRP.RotVelocity = Vector3.new(9e8, 9e8, 9e8)
        end
        task.wait()  
    until not tHRP or tick() > startTime + 1.5
    
    bv:Destroy()
    myHRP.CFrame = savedPos
    myHRP.Velocity = Vector3.new(0,0,0); myHRP.RotVelocity = Vector3.new(0,0,0)
end

-- GHOST FLING (BYPASS DE ANTI-FLING)
local function executeGhostFling(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    
    local savedPos = hrp.CFrame
    local bv = Instance.new("BodyVelocity", hrp)
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
    
    local bav = Instance.new("BodyAngularVelocity", hrp)
    bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
    bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)

    local start = tick()
    while tick() - start < 1.5 do
        if not tHRP or not tHRP.Parent then break end
        -- Movimento errático para quebrar o script de defesa dele
        hrp.CFrame = tHRP.CFrame * CFrame.new(math.random(-1,1), 0, math.random(-1,1)) * CFrame.Angles(math.random(), 0, 0)
        task.wait()
    end
    
    bv:Destroy(); bav:Destroy()
    hrp.CFrame = savedPos
    hrp.Velocity = Vector3.new(0,0,0); hrp.RotVelocity = Vector3.new(0,0,0)
end

-- AUTO GRAB GUN (REVISADO)
local function grabGun(gunDrop)
    local hrp = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    if hrp and gunDrop then
        local savedPos = hrp.CFrame
        local startTime = tick()
        repeat
            hrp.CFrame = gunDrop.CFrame
            task.wait()
        until not gunDrop.Parent or tick() > startTime + 1
        hrp.CFrame = savedPos
    end
end

workspace.ChildAdded:Connect(function(child)
    if child.Name == "GunDrop" and AUTO_GRAB_GUN then grabGun(child) end
end)



-- COIN FARM SYSTEM (TWEEN SEGURO)
local coinCollected = {}
local isTweening = false

local function findCoins()
    local c = {}
    local names = {"MainCoin", "CoinVisual", "Coin", "Coin_Server"}
    for _, o in ipairs(workspace:GetDescendants()) do
        if o:IsA("BasePart") and table.find(names, o.Name) then
            if o.Parent and not coinCollected[o:GetDebugId()] then table.insert(c, o) end
        end
    end
    return c
end

local function safeTeleport(target)
    local hrp = lp.Character and lp.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not target.Parent then return end
    isTweening = true
    local time = (hrp.Position - target.Position).Magnitude / FARM_SPEED
    local tween = TweenService:Create(hrp, TweenInfo.new(time, Enum.EasingStyle.Linear), {
        CFrame = CFrame.new(target.Position + Vector3.new(0, 1, 0))
    })
    tween:Play()
    tween.Completed:Connect(function() coinCollected[target:GetDebugId()] = true; isTweening = false end)
    repeat task.wait() until not isTweening
end

task.spawn(function()
    while true do
        if COIN_ENABLED and not isTweening and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
            local coins = findCoins()
            if #coins > 0 then
                table.sort(coins, function(a, b) return (lp.Character.HumanoidRootPart.Position - a.Position).Magnitude < (lp.Character.HumanoidRootPart.Position - b.Position).Magnitude end)
                safeTeleport(coins[1])
            end
        end
        task.wait(0.1)
    end
end)

-- ASSASSINO: HITBOX & KILL AURA
local function startHitbox()
    task.spawn(function()
        while HITBOX_ENABLED do
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    p.Character.HumanoidRootPart.Size = Vector3.new(HITBOX_SIZE, HITBOX_SIZE, HITBOX_SIZE)
                    p.Character.HumanoidRootPart.Transparency = 0.7
                    p.Character.HumanoidRootPart.CanCollide = false
                end
            end
            task.wait(1)
        end
    end)
end

local function startKillAura()
    task.spawn(function()
        while KILLAURA_ENABLED do
            local knife = lp.Character and lp.Character:FindFirstChild("Knife")
            if knife then
                for _, p in pairs(Players:GetPlayers()) do
                    if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                        if (lp.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude <= KILLAURA_RADIUS then
                            knife:Activate()
                        end
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end

-- XERIFE: SISTEMA DE DISPARO (CORRIGIDO)
local function shootAt(targetPart)
    local remote = (ReplicatedStorage:FindFirstChild("Entity") and ReplicatedStorage.Entity:FindFirstChild("Revolver") and ReplicatedStorage.Entity.Revolver:FindFirstChild("Shoot")) or ReplicatedStorage:FindFirstChild("ShootEvent") or ReplicatedStorage:FindFirstChild("GunEvent")
    local gun = (lp.Character:FindFirstChild("Gun") or lp.Character:FindFirstChild("Revolver") or lp.Backpack:FindFirstChild("Gun") or lp.Backpack:FindFirstChild("Revolver"))
    if remote and gun and targetPart then
        if gun.Parent ~= lp.Character then lp.Character.Humanoid:EquipTool(gun); task.wait(0.2) end
        remote:FireServer(targetPart.Position)
    end
end


-- INTERFACE - ABAS
local T1 = Window:MakeTab({"Home", ""}); local T2 = Window:MakeTab({"Inocente", ""})
local T3 = Window:MakeTab({"Assassino", ""}); local T4 = Window:MakeTab({"Xerife", ""})
local T5 = Window:MakeTab({"Troll", ""}); local T6 = Window:MakeTab({"Misc", ""})

T1:AddParagraph({"🌸 Cherry Hub v3.2", "Advanced Edition - Completo e Blindado."})

-- ABA INOCENTE
T2:AddSection({"Combate"})
T2:AddToggle({Name="ESP Global (M/X)", Default=false, Callback=function(v) ESP_ENABLED=v end})
T2:AddButton({"🔪 Fling Murderer", function() local m = nil; for _,p in pairs(Players:GetPlayers()) do if getPlayerRole(p) == "Murderer" then m = p end end; if m then executeFling(m) end end})
T2:AddToggle({Name="Auto Pegar Arma", Default=false, Callback=function(v) AUTO_GRAB_GUN = v end})
T2:AddSection({"💰 Farm de Moedas"})
T2:AddToggle({Name="Ativar Auto Farm", Default=false, Callback=function(v) COIN_ENABLED=v end})
T2:AddSlider({Name="Velocidade Farm", Min=10, Max=150, Default=60, Callback=function(v) FARM_SPEED=v end})

-- ABA ASSASSINO
T3:AddSection({"⚔️ Hitbox"})
T3:AddToggle({Name="Ativar Hitbox", Default=false, Callback=function(v) HITBOX_ENABLED=v; if v then startHitbox() end end})
T3:AddSlider({Name="Tamanho Hitbox", Min=1, Max=50, Default=15, Callback=function(v) HITBOX_SIZE=v end})
T3:AddSection({"🔥 Kill Aura"})
T3:AddToggle({Name="Ativar Kill Aura", Default=false, Callback=function(v) KILLAURA_ENABLED=v; if v then startKillAura() end end})
T3:AddSlider({Name="Raio Aura", Min=1, Max=50, Default=15, Callback=function(v) KILLAURA_RADIUS=v end})

-- ABA XERIFE
T4:AddSection({"🔫 Funções Xerife"})
T4:AddToggle({Name="Aimbot (Auto Atirar)", Default=false, Callback=function(v) AIMBOT_ENABLED=v; task.spawn(function() while AIMBOT_ENABLED do local m = nil; for _,p in pairs(Players:GetPlayers()) do if getPlayerRole(p) == "Murderer" then m = p end end; if m and m.Character then shootAt(m.Character.HumanoidRootPart) end; task.wait(0.5) end end) end})
T4:AddButton({"Kill Murderer (Instant)", function() local m = nil; for _,p in pairs(Players:GetPlayers()) do if getPlayerRole(p) == "Murderer" then m = p end end; if m and m.Character then shootAt(m.Character.HumanoidRootPart) end end})

-- ABA TROLL (DROPDOWN DINÂMICO)
T5:AddSection({"🎯 Selecionar Alvo"})
local pNames = function() local n = {}; for _,p in pairs(Players:GetPlayers()) do if p ~= lp then table.insert(n, p.Name) end end; return n end
local pDropdown = T5:AddDropdown({Name="Escolher Player", Options=pNames(), Default="", Callback=function(v) selectedPlayer=Players:FindFirstChild(v) end})
Players.PlayerAdded:Connect(function() pDropdown:SetOptions(pNames()) end)
Players.PlayerRemoving:Connect(function() pDropdown:SetOptions(pNames()) end)

T5:AddSection({"🛠️ Funções em Desenvolvimento"})
T5:AddButton({"👻 Ghost Fling", function() if selectedPlayer then executeGhostFling(selectedPlayer) end end})
T5:AddParagraph({"⚠️ Aviso", "GhostFling está em Desenvolvimento, é robusto contra anti-fling, pode haver bugs."})
T5:AddToggle({Name="Anti-Fling Robusto", Default=true, Callback=function(v) _G.AntiFlingEnabled = v end})
T5:AddButton({"🌀 Bug: Atravessar Paredes", function() if lp.Character then for _,p in pairs(lp.Character:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = false end end; task.wait(1.5); for _,p in pairs(lp.Character:GetDescendants()) do if p:IsA("BasePart") then p.CanCollide = true end end end end})

-- ABA MISC
T6:AddSlider({Name="Velocidade", Min=16, Max=150, Default=16, Callback=function(v) if lp.Character then lp.Character.Humanoid.WalkSpeed = v end end})
T6:AddSlider({Name="Pulo", Min=50, Max=300, Default=50, Callback=function(v) if lp.Character then lp.Character.Humanoid.JumpPower = v end end})

Window:SelectTab(T1)
print("Cherry Hub v3.2 Carregado com Sucesso!")


