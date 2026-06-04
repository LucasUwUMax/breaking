-- CARREGAMENTO DA BIBLIOTECA
local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/minhdepzai-v/LibraryRobloc/refs/heads/main/RedzLibrary.lua"))()

-- CRIAÇÃO DA JANELA PRINCIPAL
local Window = redzlib:MakeWindow({
    Title = "Cherry Hub",
    SubTitle = "v3.3 - ULTIMATE EDITION",
    SaveFolder = "CherryMM2_Ultimate"
})

-- BOTÃO DE MINIMIZAR CUSTOMIZADO
Window:AddMinimizeButton({
    Button = { Image = "rbxassetid://78702423919944", BackgroundTransparency = 0 },
    Corner = { CornerRadius = UDim.new(35, 1) },
})

-- REFERÊNCIAS DO SISTEMA
local lp = game.Players.LocalPlayer
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local TweenService = game:GetService("TweenService")
local ReplicatedStorage = game:GetService("ReplicatedStorage")

-- ESTADOS GLOBAIS (CONTROLE DE FUNÇÕES)
local ESP_ENABLED      = false
local HITBOX_ENABLED   = false
local KILLAURA_ENABLED = false
local COIN_ENABLED     = false
local FARM_SPEED       = 60
local HITBOX_SIZE      = 15
local KILLAURA_RADIUS  = 15
local AUTO_GRAB_GUN    = false
local AIMBOT_ENABLED   = false
local viewEnabled      = false
local loopGhostEnabled = false
local godModeEnabled   = false
local selectedPlayer   = nil

-- CONFIGURAÇÃO DE ANTI-FLING (PROTEÇÃO)
_G.AntiFlingEnabled = true


-- =============================================
-- IDENTIFICAÇÃO DE PAPÉIS (DETECÇÃO PROFUNDA)
-- =============================================
local function getPlayerRole(p)
    if not p or not p.Parent then return "Innocent" end
    local char = p.Character
    local bp = p.Backpack
    
    -- Verifica se é Assassino (Faca na mão ou na mochila)
    if (char and char:FindFirstChild("Knife")) or (bp and bp:FindFirstChild("Knife")) then
        return "Murderer"
    -- Verifica se é Xerife (Arma/Revolver na mão ou na mochila)
    elseif (char and (char:FindFirstChild("Gun") or char:FindFirstChild("Revolver"))) or 
           (bp and (bp:FindFirstChild("Gun") or bp:FindFirstChild("Revolver"))) then
        return "Sheriff"
    end
    return "Innocent"
end

-- =============================================
-- SISTEMA DE PREDIÇÃO (PARA NÃO ERRAR O ATAQUE)
-- =============================================
local function getAttackPos(targetHRP)
    -- Calcula a posição futura baseada na velocidade do player (Antecipação)
    local predictionTime = 0.2 -- O script ataca onde o player estará em 0.2 segundos
    local velocity = targetHRP.Velocity
    -- Se o player estiver parado, a predição é 0, se estiver correndo/pulando, ela compensa
    return targetHRP.CFrame + (velocity * predictionTime)
end

-- =============================================
-- SMART ANTI-FLING (PROTEÇÃO ATIVA)
-- =============================================
task.spawn(function()
    while task.wait() do
        if _G.AntiFlingEnabled and lp.Character and lp.Character:FindFirstChild("Humanoid") then
            -- Previne que você caia ou perca o controle do personagem ao ser atingido
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    -- Se um player vier em alta velocidade (possível Fling), desativamos a colisão dele
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

-- =============================================
-- SISTEMA DE ESP HIGHLIGHT (IDENTIFICAÇÃO VISUAL)
-- =============================================
local function applyESP(p)
    if p == lp or not p.Character then return end
    local old = p.Character:FindFirstChild("CherryHighlight")
    if old then old:Destroy() end

    -- Só mostra ESP se estiver ligado ou se o player for o alvo selecionado no View
    if not ESP_ENABLED and (not viewEnabled or p ~= selectedPlayer) then return end

    local role = getPlayerRole(p)
    if role == "Innocent" and (not viewEnabled or p ~= selectedPlayer) then return end

    local h = Instance.new("Highlight")
    h.Name = "CherryHighlight"
    h.FillTransparency = 0.4
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



-- =============================================
-- ASSASSINO: HITBOX E KILL AURA (NADA VISUAL)
-- =============================================

-- HITBOX EXPANDIDA (FACILITA ACERTAR A FACA)
local function startHitbox()
    task.spawn(function()
        while HITBOX_ENABLED do
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = p.Character.HumanoidRootPart
                    hrp.Size = Vector3.new(HITBOX_SIZE, HITBOX_SIZE, HITBOX_SIZE)
                    hrp.Transparency = 0.75
                    hrp.CanCollide = false -- Para você passar por dentro e matar fácil
                end
            end
            task.wait(1)
        end
    end)
end

-- KILL AURA (MATA AUTOMATICAMENTE AO CHEGAR PERTO)
local function startKillAura()
    task.spawn(function()
        while KILLAURA_ENABLED do
            local knife = lp.Character and (lp.Character:FindFirstChild("Knife") or lp.Backpack:FindFirstChild("Knife"))
            if knife then
                for _, p in pairs(Players:GetPlayers()) do
                    if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                        local dist = (lp.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude
                        if dist <= KILLAURA_RADIUS then
                            -- Equipa a faca se não estiver na mão
                            if knife.Parent ~= lp.Character then lp.Character.Humanoid:EquipTool(knife) end
                            knife:Activate() -- Aciona a faca no servidor
                        end
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end

-- =============================================
-- FARM DE MOEDAS (SISTEMA DE TWEEN SEGURO)
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
    -- Velocidade calculada para evitar detecção (FARM_SPEED)
    local dist = (hrp.Position - target.Position).Magnitude
    local time = dist / FARM_SPEED
    local tween = TweenService:Create(hrp, TweenInfo.new(time, Enum.EasingStyle.Linear), {CFrame = CFrame.new(target.Position + Vector3.new(0, 1, 0))})
    tween:Play()
    tween.Completed:Connect(function() 
        coinCollected[target:GetDebugId()] = true
        isTweening = false 
    end)
    repeat task.wait() until not isTweening
end

task.spawn(function()
    while true do
        if COIN_ENABLED and not isTweening and lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
            local coins = findCoins()
            if #coins > 0 then
                -- Ordena moedas pela distância (pega a mais próxima primeiro)
                table.sort(coins, function(a, b) 
                    return (lp.Character.HumanoidRootPart.Position - a.Position).Magnitude < (lp.Character.HumanoidRootPart.Position - b.Position).Magnitude 
                end)
                safeTeleport(coins[1])
            end
        end
        task.wait(0.2)
    end
end)



-- =============================================
-- XERIFE: SISTEMA DE TIRO REAL (SERVER-SIDE)
-- =============================================
local function shootAt(targetPart)
    if not targetPart then return end
    
    -- Localiza o RemoteEvent de disparo no ReplicatedStorage (MM2 muda nomes às vezes)
    local remote = nil
    for _, v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and (v.Name:find("Shoot") or v.Name:find("Gun") or v.Name:find("Fire")) then
            remote = v; break
        end
    end
    
    local gun = (lp.Character:FindFirstChild("Gun") or lp.Character:FindFirstChild("Revolver") or lp.Backpack:FindFirstChild("Gun") or lp.Backpack:FindFirstChild("Revolver"))
    
    if remote and gun then
        -- Saque da arma se necessário
        if gun.Parent ~= lp.Character then 
            lp.Character.Humanoid:EquipTool(gun)
            task.wait(0.4) -- Tempo para o servidor registrar o saque
        end
        -- Envia o tiro real para o servidor atingir o alvo
        remote:FireServer(targetPart.Position)
    end
end

-- =============================================
-- MOTOR DE GHOST FLING (TECNOLOGIA MAGNET)
-- =============================================
local function executeGhostFling(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    
    local savedPos = hrp.CFrame
    -- Configura velocidade e rotação de impacto (Física Extrema)
    local bv = Instance.new("BodyVelocity", hrp); bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
    local bav = Instance.new("BodyAngularVelocity", hrp); bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge); bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
    
    -- MAGNET: Expande a hitbox do alvo apenas para o SEU personagem, garantindo que o impacto ocorra
    local oldSize = tHRP.Size
    tHRP.Size = Vector3.new(12, 12, 12)
    tHRP.CanCollide = true

    local start = tick()
    while tick() - start < 1.5 do
        if not tHRP or not tHRP.Parent then break end
        -- Persegue a posição futura do alvo (Predição)
        hrp.CFrame = getAttackPos(tHRP) * CFrame.new(0, -0.6, 0)
        task.wait()
    end
    
    -- Restaura estado original do alvo e personagem
    tHRP.Size = oldSize
    bv:Destroy(); bav:Destroy()
    hrp.CFrame = savedPos
    hrp.Velocity = Vector3.new(0,0,0); hrp.RotVelocity = Vector3.new(0,0,0)
end

-- VARIANTES DO GHOST FLING
local function ghostFlingAll()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then executeGhostFling(p); task.wait(0.1) end
    end
end

-- AUTO PEGAR ARMA (RESTAURADO)
workspace.ChildAdded:Connect(function(child)
    if child.Name == "GunDrop" and AUTO_GRAB_GUN then
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            local s = hrp.CFrame
            repeat hrp.CFrame = child.CFrame; task.wait() until not child.Parent
            hrp.CFrame = s
        end
    end
end)





-- =============================================
-- ATAQUES GOD MODE (FÍSICA AVANÇADA)
-- =============================================

-- ORBIT FLING (CRIA UM VÁCUO AO REDOR DO ALVO)
local function executeOrbitFling(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    
    local savedPos = hrp.CFrame
    local angle = 0
    local bv = Instance.new("BodyVelocity", hrp)
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Velocity = Vector3.new(0,0,0)
    
    local start = tick()
    while tick() - start < 2.5 do
        if not tHRP or not tHRP.Parent then break end
        angle = angle + 60 -- Velocidade de rotação da órbita
        -- O personagem gira ao redor do alvo em alta velocidade, puxando-o para o vácuo
        hrp.CFrame = tHRP.CFrame * CFrame.Angles(0, math.rad(angle), 0) * CFrame.new(6, 0, 0)
        hrp.Velocity = Vector3.new(9e8, 9e8, 9e8)
        task.wait()
    end
    bv:Destroy()
    hrp.CFrame = savedPos
    hrp.Velocity = Vector3.new(0,0,0)
end

-- BLINK KILL (ATAQUE INSTANTÂNEO DE ALTA VELOCIDADE)
local function executeBlinkKill(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if hrp and tHRP then
        local savedPos = hrp.CFrame
        -- Teleporta, aplica força massiva em 1 frame e volta
        hrp.CFrame = tHRP.CFrame
        hrp.Velocity = Vector3.new(9e9, 9e9, 9e9)
        task.wait(0.06) 
        hrp.CFrame = savedPos
        hrp.Velocity = Vector3.new(0,0,0)
    end
end

-- DESYNC ATTACK (QUEBRA ANTI-FLINGS POR VIBRAÇÃO)
local function executeDesyncAttack(target)
    if not target or not target.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    local tHRP = target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    
    local savedPos = hrp.CFrame
    local start = tick()
    while tick() - start < 1.5 do
        if not tHRP or not tHRP.Parent then break end
        -- Alterna rapidamente a posição para glitchar a detecção de colisão do alvo
        hrp.CFrame = tHRP.CFrame * CFrame.new(0, 0.7, 0); task.wait()
        hrp.CFrame = tHRP.CFrame * CFrame.new(0, -0.7, 0); task.wait()
    end
    hrp.CFrame = savedPos
    hrp.Velocity = Vector3.new(0,0,0)
end

-- =============================================
-- SISTEMA DE VIEW PLAYER (PERSEGUIÇÃO VISUAL)
-- =============================================
local function toggleView(state)
    viewEnabled = state
    if viewEnabled and selectedPlayer and selectedPlayer.Character then
        local hum = selectedPlayer.Character:FindFirstChild("Humanoid")
        if hum then
            workspace.CurrentCamera.CameraSubject = hum
            applyESP(selectedPlayer) -- Garante que o alvo tenha ESP enquanto você olha
        end
    else
        -- Volta a câmera para o seu personagem
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            workspace.CurrentCamera.CameraSubject = lp.Character.Humanoid
        end
    end
end

-- Atualização contínua da câmera se o View estiver ligado
RunService.RenderStepped:Connect(function()
    if viewEnabled and selectedPlayer and selectedPlayer.Character then
        local hum = selectedPlayer.Character:FindFirstChild("Humanoid")
        if hum then workspace.CurrentCamera.CameraSubject = hum end
    end
end)



-- =============================================
-- CRIAÇÃO DAS ABAS (NAVEGAÇÃO)
-- =============================================
local T1 = Window:MakeTab({"Home", ""})
local T2 = Window:MakeTab({"Inocente", ""})
local T3 = Window:MakeTab({"Assassino", ""})
local T4 = Window:MakeTab({"Xerife", ""})
local T5 = Window:MakeTab({"Troll", ""})
local T6 = Window:MakeTab({"Misc", ""})

-- ABA HOME (BOAS-VINDAS)
T1:AddParagraph({"🌸 Cherry Hub v3.3", "ULTIMATE EDITION - Poder Total Restaurado e Blindado."})
T1:AddParagraph({"Dica de Uso", "Use o Ghost Fling na aba Troll para eliminar players que correm ou pulam."})

-- ABA INOCENTE (DEFESA E FARM)
T2:AddSection({"🛡️ Defesa e ESP"})
T2:AddToggle({Name="ESP Global (Players)", Default=false, Callback=function(v) ESP_ENABLED = v end})
T2:AddToggle({Name="Auto Pegar Arma", Default=false, Callback=function(v) AUTO_GRAB_GUN = v end})

T2:AddSection({"💰 Farm de Moedas"})
T2:AddToggle({Name="Ativar Coin Farm (Tween)", Default=false, Callback=function(v) COIN_ENABLED = v end})
T2:AddSlider({Name="Velocidade do Farm", Min=30, Max=150, Default=60, Callback=function(v) FARM_SPEED = v end})

-- ABA ASSASSINO (COMBATE)
T3:AddSection({"⚔️ Hitbox do Mal"})
T3:AddToggle({Name="Ativar Hitbox", Default=false, Callback=function(v) HITBOX_ENABLED = v; if v then startHitbox() end end})
T3:AddSlider({Name="Tamanho Hitbox", Min=10, Max=60, Default=25, Callback=function(v) HITBOX_SIZE = v end})

T3:AddSection({"🔥 Kill Aura (Magnet)"})
T3:AddToggle({Name="Ativar Kill Aura", Default=false, Callback=function(v) KILLAURA_ENABLED = v; if v then startKillAura() end end})
T3:AddSlider({Name="Raio da Aura", Min=5, Max=50, Default=20, Callback=function(v) KILLAURA_RADIUS = v end})

-- ABA XERIFE (TIRO REAL)
T4:AddSection({"🔫 Funções de Xerife"})
T4:AddToggle({Name="Silent Aim (Auto Atirar)", Default=false, Callback=function(v) 
    AIMBOT_ENABLED = v
    task.spawn(function()
        while AIMBOT_ENABLED do
            -- Busca automática pelo Murderer para atirar
            local target = nil
            for _,p in pairs(Players:GetPlayers()) do
                if getPlayerRole(p) == "Murderer" then target = p; break end
            end
            if target and target.Character then 
                shootAt(target.Character:FindFirstChild("HumanoidRootPart")) 
            end
            task.wait(0.7) -- Delay para o servidor registrar o dano
        end
    end)
end})

T4:AddButton({"Matar Murderer Agora", function() 
    local target = nil
    for _,p in pairs(Players:GetPlayers()) do
        if getPlayerRole(p) == "Murderer" then target = p; break end
    end
    if target and target.Character then 
        shootAt(target.Character:FindFirstChild("HumanoidRootPart")) 
    end
end})



-- =============================================
-- ABA TROLL: SELETOR E ATAQUES AVANÇADOS
-- =============================================

T5:AddSection({"🎯 Selecionar Alvo"})

-- DROPDOWN DINÂMICO DE JOGADORES
local function getPlayerNames()
    local names = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp then table.insert(names, p.Name) end
    end
    return names
end

local pDropdown = T5:AddDropdown({
    Name = "Escolher Alvo",
    Options = getPlayerNames(),
    Default = "",
    Callback = function(v)
        selectedPlayer = Players:FindFirstChild(v)
    end
})

-- Atualiza a lista de players automaticamente quando alguém entra ou sai
Players.PlayerAdded:Connect(function() pDropdown:SetOptions(getPlayerNames()) end)
Players.PlayerRemoving:Connect(function() pDropdown:SetOptions(getPlayerNames()) end)

T5:AddButton({"🔪 Normal Fling", function() 
    if selectedPlayer then executeFling(selectedPlayer) end 
end})

-- =============================================
-- SEÇÃO: FUNÇÕES EM DESENVOLVIMENTO (GOD MODE)
-- =============================================
T5:AddSection({"🛠️ Funções em Desenvolvimento"})

-- GHOST FLING UNITÁRIO
T5:AddButton({"👻 Ghost Fling (Magnet)", function() 
    if selectedPlayer then executeGhostFling(selectedPlayer) end 
end})

-- LOOP GHOST FLING (PERSEGUIÇÃO CONTÍNUA)
T5:AddToggle({
    Name = "Loop Ghost Fling", 
    Default = false, 
    Callback = function(v) 
        loopGhostEnabled = v
        task.spawn(function()
            while loopGhostEnabled do
                if selectedPlayer then executeGhostFling(selectedPlayer) end
                task.wait(0.5)
            end
        end)
    end
})

-- VARIANTES AUTOMÁTICAS DE GHOST FLING
T5:AddButton({"💀 Ghost Fling: Murderer", function() 
    local m = getTargetByRole("Murderer")
    if m then executeGhostFling(m) end
end})

T5:AddButton({"👮 Ghost Fling: Sheriff", function() 
    local s = getTargetByRole("Sheriff")
    if s then executeGhostFling(s) end
end})

T5:AddButton({"🌪️ Ghost Fling: ALL (Server Wipe)", function() 
    ghostFlingAll() 
end})

-- NOVOS MÉTODOS DE FÍSICA
T5:AddButton({"🪐 Orbit Fling (Vácuo)", function() 
    if selectedPlayer then executeOrbitFling(selectedPlayer) end 
end})

T5:AddButton({"⚡ Blink Kill (Impacto)", function() 
    if selectedPlayer then executeBlinkKill(selectedPlayer) end 
end})

T5:AddButton({"📳 Desync Attack (Anti-AntiFling)", function() 
    if selectedPlayer then executeDesyncAttack(selectedPlayer) end 
end})

-- CONTROLE DE CÂMERA (VIEW)
T5:AddToggle({
    Name = "👀 View Player", 
    Default = false, 
    Callback = function(v) 
        toggleView(v)
    end
})

T5:AddParagraph({"Aviso de Poder", "As funções acima usam Magnet e Predição. Se o alvo pular ou correr, o script irá antecipar o movimento e expandir a hitbox de colisão para garantir o Fling."})




-- =============================================
-- ABA MISC: VELOCIDADE E GOD MODE
-- =============================================

T6:AddSection({"⚡ Movimentação e Física"})

-- WALK SPEED SLIDER (VELOCIDADE REAL)
T6:AddSlider({
    Name = "Velocidade de Caminhada", 
    Min = 16, 
    Max = 200, 
    Default = 16, 
    Callback = function(v) 
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            lp.Character.Humanoid.WalkSpeed = v
        end
    end
})

-- JUMP POWER SLIDER
T6:AddSlider({
    Name = "Poder do Pulo", 
    Min = 50, 
    Max = 300, 
    Default = 50, 
    Callback = function(v) 
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            lp.Character.Humanoid.JumpPower = v
        end
    end
})

T6:AddSection({"🛡️ Proteções Especiais"})

-- GOD MODE (PHYSICS DESYNC)
T6:AddToggle({
    Name = "God Mode (Desync)", 
    Default = false, 
    Callback = function(v) 
        godModeEnabled = v
        task.spawn(function()
            while godModeEnabled do
                if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = lp.Character.HumanoidRootPart
                    local oldCFrame = hrp.CFrame
                    -- O "Desync": Move seu personagem para longe e volta instantaneamente
                    -- O servidor não consegue processar o dano na posição correta
                    hrp.CFrame = oldCFrame * CFrame.new(0, 50, 0)
                    task.wait()
                    hrp.CFrame = oldCFrame
                end
                task.wait(0.05)
            end
        end)
    end
})

-- ANTI-FLING TOGGLE
T6:AddToggle({
    Name = "Anti-Fling Ativo", 
    Default = true, 
    Callback = function(v) 
        _G.AntiFlingEnabled = v 
    end
})

T6:AddSection({"⚙️ Script"})
T6:AddButton({"Destruir Interface", function() 
    redzlib:Destroy() 
end})

-- SELECIONA A ABA INICIAL
Window:SelectTab(T1)

-- MENSAGEM DE CARREGAMENTO NO CONSOLE DO EXECUTOR
print([[
-------------------------------------------
   CHERRY HUB v3.3 ULTIMATE LOADED!
   - Magnet Physics: ON
   - Prediction Engine: ON
   - Real-Time Role Identification: ON
-------------------------------------------
]])



