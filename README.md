local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/minhdepzai-v/LibraryRobloc/refs/heads/main/RedzLibrary.lua"))()

local Window = redzlib:MakeWindow({
    Title = "Cherry Hub",
    SubTitle = "v4.7 - God Mode Edition",
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

-- CONFIGURAÇÕES GLOBAIS
local ESP_ENABLED      = false
local HITBOX_ENABLED   = false
local KILLAURA_ENABLED = false
local COIN_ENABLED     = false
local FARM_SPEED       = 60
local HITBOX_SIZE      = 15
local KILLAURA_RADIUS  = 15

local selectedPlayer   = nil
local viewEnabled      = false
local loopGhostEnabled = false
local AUTO_GRAB_GUN    = false
local AIMBOT_ENABLED   = false
_G.AntiFlingEnabled    = true

-- NOVO: SMART ANTI-FLING ROBUSTO (PROTEÇÃO ATIVA)
task.spawn(function()
    while task.wait() do
        if _G.AntiFlingEnabled and lp.Character and lp.Character:FindFirstChild("Humanoid") then
            -- Previne queda por colisão de alta velocidade
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    -- Detecta scripters vindo em alta velocidade e desativa a colisão deles
                    if hrp and (hrp.Velocity.Magnitude > 45 or hrp.RotVelocity.Magnitude > 45) then
                        for _, part in pairs(p.Character:GetDescendants()) do
                            if part:IsA("BasePart") then part.CanCollide = false end
                        end
                    end
                end
            end
        end
    end
end)




-- SISTEMA DE IDENTIFICAÇÃO DE PAPÉIS (BUSCA PROFUNDA)
local function getPlayerRole(p)
    if not p or not p.Parent then return "Innocent" end
    local char = p.Character
    local bp = p.Backpack
    -- Verifica Faca (Murderer)
    if (char and char:FindFirstChild("Knife")) or (bp and bp:FindFirstChild("Knife")) then
        return "Murderer"
    -- Verifica Arma (Sheriff)
    elseif (char and (char:FindFirstChild("Gun") or char:FindFirstChild("Revolver"))) or 
           (bp and (bp:FindFirstChild("Gun") or bp:FindFirstChild("Revolver"))) then
        return "Sheriff"
    end
    return "Innocent"
end

-- SISTEMA DE ESP (HIGHLIGHT REAL-TIME)
local function applyESP(p)
    if p == lp or not p.Character then return end
    local old = p.Character:FindFirstChild("CherryHighlight")
    if old then old:Destroy() end

    if not ESP_ENABLED and (not viewEnabled or p ~= selectedPlayer) then return end

    local role = getPlayerRole(p)
    if role == "Innocent" and (not viewEnabled or p ~= selectedPlayer) then return end

    local h = Instance.new("Highlight")
    h.Name = "CherryHighlight"
    h.FillTransparency = 0.4
    h.OutlineTransparency = 0
    h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    h.Parent = p.Character

    if role == "Murderer" then h.FillColor = Color3.fromRGB(255, 0, 0)
    elseif role == "Sheriff" then h.FillColor = Color3.fromRGB(0, 150, 255)
    else h.FillColor = Color3.fromRGB(255, 255, 255) end
end

task.spawn(function()
    while task.wait(1) do
        for _, p in pairs(Players:GetPlayers()) do applyESP(p) end
    end
end)

-- LÓGICA DE PREDIÇÃO (PARA ALVOS EM MOVIMENTO/PULO)
local function getPredictedCFrame(targetHRP)
    local predictionTime = 0.18 -- Antecipação de 0.18s
    return targetHRP.CFrame + (targetHRP.Velocity * predictionTime)
end

-- FUNÇÃO AUXILIAR PARA PEGAR ALVOS POR CARGO
local function getTargetByRole(role)
    for _, p in pairs(Players:GetPlayers()) do
        if getPlayerRole(p) == role then return p end
    end
    return nil
end



-- =============================================
-- MOTORES DE ATAQUE (PHYSICS MANIPULATION)
-- =============================================

-- GHOST FLING (GOD MODE - PENETRAÇÃO)
local function executeGhostFling(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    
    local savedPos = hrp.CFrame
    local bv = Instance.new("BodyVelocity", hrp); bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
    local bav = Instance.new("BodyAngularVelocity", hrp); bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge); bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
    
    local start = tick()
    while tick() - start < 1.3 do
        if not tHRP or not tHRP.Parent then break end
        -- O SEGREDO: Uso da predição para entrar no corpo do alvo mesmo se ele pular
        hrp.CFrame = getPredictedCFrame(tHRP) * CFrame.new(0, -0.6, 0)
        task.wait()
    end
    bv:Destroy(); bav:Destroy(); hrp.CFrame = savedPos; hrp.Velocity = Vector3.new(0,0,0); hrp.RotVelocity = Vector3.new(0,0,0)
end

-- ORBIT FLING (VÁCUO GRAVITACIONAL)
local function executeOrbitFling(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    
    local savedPos = hrp.CFrame; local angle = 0
    local bv = Instance.new("BodyVelocity", hrp); bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.new(0,0,0)
    
    local start = tick()
    while tick() - start < 2 do
        if not tHRP or not tHRP.Parent then break end
        angle = angle + 50 -- Velocidade da órbita
        hrp.CFrame = tHRP.CFrame * CFrame.Angles(0, math.rad(angle), 0) * CFrame.new(5, 0, 0)
        hrp.Velocity = Vector3.new(9e8, 9e8, 9e8)
        task.wait()
    end
    bv:Destroy(); hrp.CFrame = savedPos; hrp.Velocity = Vector3.new(0,0,0)
end

-- DESYNC ATTACK (QUEBRA ANTI-FLING POR VIBRAÇÃO)
local function executeDesyncAttack(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    
    local savedPos = hrp.CFrame
    local start = tick()
    while tick() - start < 1.5 do
        if not tHRP or not tHRP.Parent then break end
        -- Vibração extrema para "bugar" a detecção de colisão do alvo
        hrp.CFrame = tHRP.CFrame * CFrame.new(0, 0.7, 0); task.wait()
        hrp.CFrame = tHRP.CFrame * CFrame.new(0, -0.7, 0); task.wait()
    end
    hrp.CFrame = savedPos; hrp.Velocity = Vector3.new(0,0,0)
end

-- BLINK KILL (TELEPORTE DE IMPACTO)
local function executeBlinkKill(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if hrp and tHRP then
        local savedPos = hrp.CFrame
        hrp.CFrame = tHRP.CFrame
        hrp.Velocity = Vector3.new(9e9, 9e9, 9e9)
        task.wait(0.05) -- Janela de impacto real
        hrp.CFrame = savedPos; hrp.Velocity = Vector3.new(0,0,0)
    end
end



-- =============================================
-- COIN FARM (TWEEN SISTEMAS)
-- =============================================
local coinCollected = {}
local isTweening = false

local function findCoins()
    local c = {}
    for _, o in ipairs(workspace:GetDescendants()) do
        if o:IsA("BasePart") and (o.Name == "MainCoin" or o.Name == "Coin") then
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
    local tween = TweenService:Create(hrp, TweenInfo.new(time, Enum.EasingStyle.Linear), {CFrame = CFrame.new(target.Position + Vector3.new(0, 1, 0))})
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

-- =============================================
-- ASSASSINO (HITBOX & KILL AURA)
-- =============================================
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

-- =============================================
-- XERIFE BLINDADO (FIRESERVER FIX)
-- =============================================
local function shootAt(targetPart)
    local remote = nil
    for _, v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and (v.Name == "Shoot" or v.Name == "GunEvent" or v.Name == "ShootEvent") then
            remote = v; break
        end
    end
    local gun = (lp.Character:FindFirstChild("Gun") or lp.Character:FindFirstChild("Revolver") or lp.Backpack:FindFirstChild("Gun") or lp.Backpack:FindFirstChild("Revolver"))
    if remote and gun and targetPart then
        if gun.Parent ~= lp.Character then lp.Character.Humanoid:EquipTool(gun); task.wait(0.3) end
        remote:FireServer(targetPart.Position)
    end
end



-- =============================================
-- INTERFACE GRÁFICA (TABS)
-- =============================================
local T1 = Window:MakeTab({"Home", ""}); local T2 = Window:MakeTab({"Inocente", ""})
local T3 = Window:MakeTab({"Assassino", ""}); local T4 = Window:MakeTab({"Xerife", ""})
local T5 = Window:MakeTab({"Troll", ""}); local T6 = Window:MakeTab({"Misc", ""})

T1:AddParagraph({"🌸 Cherry Hub v3.2", "God Mode Edition - O script mais potente de MM2."})

-- ABA INOCENTE
T2:AddSection({"Combate & ESP"})
T2:AddToggle({Name="ESP Global (M/X)", Default=false, Callback=function(v) ESP_ENABLED=v end})
T2:AddToggle({Name="Auto Pegar Arma", Default=false, Callback=function(v) AUTO_GRAB_GUN = v end})
T2:AddSection({"💰 Farm"})
T2:AddToggle({Name="Ativar Auto Farm", Default=false, Callback=function(v) COIN_ENABLED=v end})

-- ABA ASSASSINO
T3:AddSection({"⚔️ Hitbox"})
T3:AddToggle({Name="Ativar Hitbox", Default=false, Callback=function(v) HITBOX_ENABLED=v; if v then startHitbox() end end})
T3:AddSlider({Name="Tamanho Hitbox", Min=1, Max=50, Default=15, Callback=function(v) HITBOX_SIZE=v end})
T3:AddSection({"🔥 Kill Aura"})
T3:AddToggle({Name="Ativar Kill Aura", Default=false, Callback=function(v) KILLAURA_ENABLED=v; if v then startKillAura() end end})

-- ABA XERIFE (CORRIGIDA)
T4:AddSection({"🔫 Funções Xerife"})
T4:AddToggle({Name="Aimbot (Auto Atirar)", Default=false, Callback=function(v) AIMBOT_ENABLED=v; task.spawn(function() while AIMBOT_ENABLED do local m=getTargetByRole("Murderer"); if m and m.Character then shootAt(m.Character.HumanoidRootPart) end; task.wait(0.8) end end) end})
T4:AddButton({"Kill Murderer (Instant)", function() local m=getTargetByRole("Murderer"); if m and m.Character then shootAt(m.Character.HumanoidRootPart) end end})

-- ABA TROLL (DROPDOWN DINÂMICO)
T5:AddSection({"🎯 Selecionar Alvo"})
local pNames = function() local n = {}; for _,p in pairs(Players:GetPlayers()) do if p ~= lp then table.insert(n, p.Name) end end; return n end
local pDropdown = T5:AddDropdown({Name="Escolher Player", Options=pNames(), Default="", Callback=function(v) selectedPlayer=Players:FindFirstChild(v) end})
Players.PlayerAdded:Connect(function() pDropdown:SetOptions(pNames()) end)
Players.PlayerRemoving:Connect(function() pDropdown:SetOptions(pNames()) end)

T5:AddButton({"🔪 Normal Fling", function() if selectedPlayer then executeFling(selectedPlayer) end end})

-- SEÇÃO: FUNÇÕES EM DESENVOLVIMENTO (ARSENAL GOD MODE)
T5:AddSection({"🛠️ Funções em Desenvolvimento"})

T5:AddButton({"👻 Ghost Fling (Alvo)", function() if selectedPlayer then executeGhostFling(selectedPlayer) end end})
T5:AddToggle({Name="Loop Ghost Fling", Default=false, Callback=function(v) loopGhostEnabled=v; task.spawn(function() while loopGhostEnabled do if selectedPlayer then executeGhostFling(selectedPlayer) end; task.wait(0.5) end end) end})

T5:AddButton({"💀 Ghost Fling: Murderer", function() local m = getTargetByRole("Murderer"); if m then executeGhostFling(m) end end})
T5:AddButton({"👮 Ghost Fling: Sheriff", function() local s = getTargetByRole("Sheriff"); if s then executeGhostFling(s) end end})
T5:AddButton({"🌪️ Ghost Fling: ALL (Server Wipe)", function() for _,p in pairs(Players:GetPlayers()) do if p ~= lp then executeGhostFling(p); task.wait(0.1) end end end})

T5:AddButton({"🪐 Orbit Fling", function() if selectedPlayer then executeOrbitFling(selectedPlayer) end end})
T5:AddButton({"⚡ Blink Kill", function() if selectedPlayer then executeBlinkKill(selectedPlayer) end end})
T5:AddButton({"📳 Desync Attack", function() if selectedPlayer then executeDesyncAttack(selectedPlayer) end end})

T5:AddToggle({Name="👀 View Player", Default=false, Callback=function(v) viewEnabled=v; if v then RunService:BindToRenderStep("ViewL", 100, function() if viewEnabled and selectedPlayer and selectedPlayer.Character then workspace.CurrentCamera.CameraSubject = selectedPlayer.Character:FindFirstChild("Humanoid") else workspace.CurrentCamera.CameraSubject = lp.Character:FindFirstChild("Humanoid") end end) else RunService:UnbindFromRenderStep("ViewL"); workspace.CurrentCamera.CameraSubject = lp.Character:FindFirstChild("Humanoid") end end})

-- ABA MISC
T6:AddSection({"⚡ Movimentação"})
T6:AddSlider({Name="Velocidade", Min=16, Max=150, Default=16, Callback=function(v) if lp.Character then lp.Character.Humanoid.WalkSpeed = v end end})
T6:AddToggle({Name="Anti-Fling Robusto", Default=true, Callback=function(v) _G.AntiFlingEnabled = v end})

Window:SelectTab(T1)
print("Cherry Hub v3.2 God Mode Edition Loaded!")
