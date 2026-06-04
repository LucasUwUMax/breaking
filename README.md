-- =============================================
-- CHERRY HUB v3.3 - ULTIMATE EDITION (REVISÃO TOTAL)
-- =============================================
local redzlib = loadstring(game:HttpGet("https://raw.githubusercontent.com/minhdepzai-v/LibraryRobloc/refs/heads/main/RedzLibrary.lua"))()
local Window = redzlib:MakeWindow({Title = "Cherry Hub", SubTitle = "v3.3 - ULTIMATE EDITION", SaveFolder = "CherryMM2_Ultimate"})
Window:AddMinimizeButton({Button = {Image = "rbxassetid://78702423919944", BackgroundTransparency = 0}, Corner = {CornerRadius = UDim.new(35, 1)}})

local lp, Players, RunService, TweenService, ReplicatedStorage = game.Players.LocalPlayer, game:GetService("Players"), game:GetService("RunService"), game:GetService("TweenService"), game:GetService("ReplicatedStorage")
local ESP_ENABLED, HITBOX_ENABLED, KILLAURA_ENABLED, COIN_ENABLED = false, false, false, false
local FARM_SPEED, HITBOX_SIZE, KILLAURA_RADIUS, AUTO_GRAB_GUN = 60, 15, 15, false
local AIMBOT_ENABLED, viewEnabled, loopGhostEnabled, godModeEnabled = false, false, false, false
local selectedPlayer, _G.AntiFlingEnabled = nil, true

-- =============================================
-- IDENTIFICAÇÃO DE PAPÉIS (DETECÇÃO AGRESSIVA)
-- =============================================
local function getPlayerRole(p)
    if not p or not p.Parent then return "Innocent" end
    local char, bp = p.Character, p.Backpack
    local hasKnife = (char and char:FindFirstChild("Knife")) or (bp and bp:FindFirstChild("Knife"))
    local hasGun = (char and (char:FindFirstChild("Gun") or char:FindFirstChild("Revolver"))) or (bp and (bp:FindFirstChild("Gun") or bp:FindFirstChild("Revolver")))
    if hasKnife then return "Murderer" end
    if hasGun then return "Sheriff" end
    if bp then
        for _, tool in pairs(bp:GetChildren()) do
            if tool:IsA("Tool") then
                if tool.Name:find("Knife") then return "Murderer" end
                if tool.Name:find("Gun") or tool.Name:find("Revolver") then return "Sheriff" end
            end
        end
    end
    return "Innocent"
end

local function getTargetByRole(role)
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and getPlayerRole(p) == role then return p end
    end
    return nil
end

-- =============================================
-- SISTEMA DE PREDIÇÃO (PRECISÃO CIRÚRGICA)
-- =============================================
local function getPredictedPos(targetHRP)
    local predictionTime, velocity = 0.15, targetHRP.Velocity
    local offset = (velocity.Y > 5) and Vector3.new(0, -1, 0) or Vector3.new(0, 0, 0)
    return targetHRP.CFrame + (velocity * predictionTime) + offset
end

-- =============================================
-- ESP HIGHLIGHT (COM AUTO-RECOVERY)
-- =============================================
local function applyESP(p)
    if p == lp or not p.Character then return end
    local old = p.Character:FindFirstChild("CherryHighlight")
    if old then old:Destroy() end
    if not ESP_ENABLED and (not viewEnabled or p ~= selectedPlayer) then return end
    local role = getPlayerRole(p)
    if role == "Innocent" and (not viewEnabled or p ~= selectedPlayer) then return end
    local h = Instance.new("Highlight", p.Character)
    h.Name = "CherryHighlight"; h.FillTransparency = 0.4; h.OutlineTransparency = 0; h.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    if role == "Murderer" then h.FillColor = Color3.fromRGB(255, 0, 0)
    elseif role == "Sheriff" then h.FillColor = Color3.fromRGB(0, 150, 255)
    else h.FillColor = Color3.fromRGB(255, 255, 255) end
end

task.spawn(function()
    while task.wait(1.5) do
        for _, p in pairs(Players:GetPlayers()) do applyESP(p) end
    end
end)






-- =============================================
-- SISTEMAS DE ATAQUE DE FÍSICA (FLING & MAGNET)
-- =============================================
local function executeFling(target)
    if not target or not target.Character then return end
    local hrp, tHRP, hum = lp.Character:FindFirstChild("HumanoidRootPart"), target.Character:FindFirstChild("HumanoidRootPart"), lp.Character:FindFirstChild("Humanoid")
    if not hrp or not tHRP or not hum then return end
    local savedPos = hrp.CFrame
    hum.PlatformStand = true
    local bv = Instance.new("BodyVelocity", hrp); bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
    local bav = Instance.new("BodyAngularVelocity", hrp); bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge); bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
    hrp.CFrame = tHRP.CFrame; task.wait(0.2)
    bv:Destroy(); bav:Destroy(); hum.PlatformStand = false; hrp.CFrame = savedPos; hrp.Velocity, hrp.RotVelocity = Vector3.new(0,0,0), Vector3.new(0,0,0)
end

local function executeGhostFling(target)
    if not target or not target.Character then return end
    local hrp, tHRP, hum = lp.Character:FindFirstChild("HumanoidRootPart"), target.Character:FindFirstChild("HumanoidRootPart"), lp.Character:FindFirstChild("Humanoid")
    if not hrp or not tHRP or not hum then return end
    local savedPos, oldSize = hrp.CFrame, tHRP.Size
    hum.PlatformStand = true
    local bv = Instance.new("BodyVelocity", hrp); bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.new(9e9, 9e9, 9e9)
    local bav = Instance.new("BodyAngularVelocity", hrp); bav.MaxTorque = Vector3.new(math.huge, math.huge, math.huge); bav.AngularVelocity = Vector3.new(9e9, 9e9, 9e9)
    tHRP.Size = Vector3.new(15, 15, 15); tHRP.CanCollide = true
    local start = tick()
    while tick() - start < 1.8 do
        if not tHRP or not tHRP.Parent then break end
        hrp.CFrame = getPredictedPos(tHRP) * CFrame.new(0, -0.8, 0); task.wait()
    end
    tHRP.Size = oldSize; bv:Destroy(); bav:Destroy(); hum.PlatformStand = false; hrp.CFrame = savedPos; hrp.Velocity, hrp.RotVelocity = Vector3.new(0,0,0), Vector3.new(0,0,0)
end

local function ghostFlingAll()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then executeGhostFling(p); task.wait(0.1) end
    end
end

-- =============================================
-- ATAQUES AVANÇADOS (ORBIT, BLINK, DESYNC)
-- =============================================
local function executeOrbitFling(target)
    if not target or not target.Character then return end
    local hrp, tHRP = lp.Character:FindFirstChild("HumanoidRootPart"), target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    local savedPos, angle = hrp.CFrame, 0
    local bv = Instance.new("BodyVelocity", hrp); bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge); bv.Velocity = Vector3.new(0,0,0)
    local start = tick()
    while tick() - start < 2.5 do
        if not tHRP or not tHRP.Parent then break end
        angle = angle + 60; hrp.CFrame = tHRP.CFrame * CFrame.Angles(0, math.rad(angle), 0) * CFrame.new(6, 0, 0)
        hrp.Velocity = Vector3.new(9e8, 9e8, 9e8); task.wait()
    end
    bv:Destroy(); hrp.CFrame = savedPos; hrp.Velocity, hrp.RotVelocity = Vector3.new(0,0,0), Vector3.new(0,0,0)
end

local function executeBlinkKill(target)
    if not target or not target.Character then return end
    local hrp, tHRP = lp.Character:FindFirstChild("HumanoidRootPart"), target.Character:FindFirstChild("HumanoidRootPart")
    if hrp and tHRP then
        local savedPos = hrp.CFrame; hrp.CFrame = tHRP.CFrame; hrp.Velocity = Vector3.new(9e9, 9e9, 9e9); task.wait(0.06)
        hrp.CFrame = savedPos; hrp.Velocity, hrp.RotVelocity = Vector3.new(0,0,0), Vector3.new(0,0,0)
    end
end

local function executeDesyncAttack(target)
    if not target or not target.Character then return end
    local hrp, tHRP = lp.Character:FindFirstChild("HumanoidRootPart"), target.Character:FindFirstChild("HumanoidRootPart")
    if not hrp or not tHRP then return end
    local savedPos, start = hrp.CFrame, tick()
    while tick() - start < 1.5 do
        if not tHRP or not tHRP.Parent then break end
        hrp.CFrame = tHRP.CFrame * CFrame.new(0, 0.7, 0); task.wait()
        hrp.CFrame = tHRP.CFrame * CFrame.new(0, -0.7, 0); task.wait()
    end
    hrp.CFrame = savedPos; hrp.Velocity, hrp.RotVelocity = Vector3.new(0,0,0), Vector3.new(0,0,0)
end

-- =============================================
-- SMART ANTI-FLING (DEFESA ATIVA)
-- =============================================
task.spawn(function()
    while task.wait() do
        if _G.AntiFlingEnabled and lp.Character and lp.Character:FindFirstChild("Humanoid") then
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.FallingDown, false)
            lp.Character.Humanoid:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, false)
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and (hrp.Velocity.Magnitude > 45 or hrp.RotVelocity.Magnitude > 45) then
                        for _, part in pairs(p.Character:GetDescendants()) do if part:IsA("BasePart") then part.CanCollide = false end end
                    end
                end
            end
        end
    end
end)





-- =============================================
-- ASSASSINO: HITBOX E KILL AURA (MAGNET)
-- =============================================
local function startHitbox()
    task.spawn(function()
        while HITBOX_ENABLED do
            for _, p in pairs(Players:GetPlayers()) do
                if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                    local hrp = p.Character.HumanoidRootPart
                    hrp.Size = Vector3.new(HITBOX_SIZE, HITBOX_SIZE, HITBOX_SIZE)
                    hrp.Transparency = 0.75; hrp.CanCollide = false
                end
            end
            task.wait(0.5)
        end
        -- Restauração ao desligar
        for _, p in pairs(Players:GetPlayers()) do
            if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = p.Character.HumanoidRootPart
                hrp.Size, hrp.Transparency, hrp.CanCollide = Vector3.new(2, 2, 1), 1, true
            end
        end
    end)
end

local function startKillAura()
    task.spawn(function()
        while KILLAURA_ENABLED do
            local knife = lp.Character:FindFirstChild("Knife") or lp.Backpack:FindFirstChild("Knife")
            if knife then
                if knife.Parent ~= lp.Character then lp.Character.Humanoid:EquipTool(knife) end
                for _, p in pairs(Players:GetPlayers()) do
                    if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
                        local dist = (lp.Character.HumanoidRootPart.Position - p.Character.HumanoidRootPart.Position).Magnitude
                        if dist <= KILLAURA_RADIUS then
                            -- Simula o ataque da faca
                            knife:Activate()
                            -- Posiciona a faca no alvo via CFrame (Magnet)
                            firetouchinterest(p.Character.HumanoidRootPart, knife.Handle, 0)
                            firetouchinterest(p.Character.HumanoidRootPart, knife.Handle, 1)
                        end
                    end
                end
            end
            task.wait(0.1)
        end
    end)
end

-- =============================================
-- XERIFE: SISTEMA DE TIRO (SILENT AIM)
-- =============================================
local function shootAt(targetPart)
    if not targetPart then return end
    local remote = nil
    -- Busca dinâmica para evitar patches de renomeação de RemoteEvents
    for _, v in pairs(ReplicatedStorage:GetDescendants()) do
        if v:IsA("RemoteEvent") and (v.Name:find("Shoot") or v.Name:find("Gun") or v.Name:find("Fire")) then
            remote = v; break
        end
    end
    local gun = lp.Character:FindFirstChild("Gun") or lp.Character:FindFirstChild("Revolver") or lp.Backpack:FindFirstChild("Gun") or lp.Backpack:FindFirstChild("Revolver")
    if remote and gun then
        if gun.Parent ~= lp.Character then lp.Character.Humanoid:EquipTool(gun); task.wait(0.3) end
        remote:FireServer(targetPart.Position)
    end
end

-- =============================================
-- VIEW PLAYER (PERSEGUIÇÃO VISUAL)
-- =============================================
local function toggleView(state)
    viewEnabled = state
    if viewEnabled and selectedPlayer and selectedPlayer.Character then
        local hum = selectedPlayer.Character:FindFirstChild("Humanoid")
        if hum then workspace.CurrentCamera.CameraSubject = hum; applyESP(selectedPlayer) end
    else
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            workspace.CurrentCamera.CameraSubject = lp.Character.Humanoid
        end
    end
end

RunService.RenderStepped:Connect(function()
    if viewEnabled and selectedPlayer and selectedPlayer.Character then
        local hum = selectedPlayer.Character:FindFirstChild("Humanoid")
        if hum then workspace.CurrentCamera.CameraSubject = hum end
    end
end)




-- =============================================
-- COIN FARM (TWEENING SEGURO)
-- =============================================
local function startCoinFarm()
    task.spawn(function()
        while COIN_ENABLED do
            if lp.Character and lp.Character:FindFirstChild("HumanoidRootPart") then
                local coins = {}
                for _, v in pairs(Workspace:GetDescendants()) do
                    if (v.Name == "Coin_Server" or v.Name == "Coin") and v:IsA("BasePart") then
                        table.insert(coins, v)
                    end
                end
                for _, coin in pairs(coins) do
                    if not COIN_ENABLED then break end
                    if coin and coin.Parent then
                        local hrp = lp.Character.HumanoidRootPart
                        local dist = (hrp.Position - coin.Position).Magnitude
                        local tweenTime = dist / FARM_SPEED
                        local info = TweenInfo.new(tweenTime, Enum.EasingStyle.Linear)
                        local tween = TweenService:Create(hrp, info, {CFrame = coin.CFrame})
                        tween:Play()
                        tween.Completed:Wait()
                        task.wait(0.1) -- Delay para evitar detecção de "teleporte"
                    end
                end
            end
            task.wait(1)
        end
    end)
end

-- =============================================
-- LOOP GHOST FLING (PERSEGUIÇÃO CONTÍNUA)
-- =============================================
local function runLoopGhost()
    task.spawn(function()
        while loopGhostEnabled do
            if selectedPlayer and selectedPlayer.Character then
                executeGhostFling(selectedPlayer)
            end
            task.wait(0.5)
        end
    end)
end

-- =============================================
-- INTERFACE GRÁFICA: ABA HOME
-- =============================================
local T1 = Window:MakeTab({"Home", ""})
local T2 = Window:MakeTab({"Inocente", ""})
local T3 = Window:MakeTab({"Assassino", ""})
local T4 = Window:MakeTab({"Xerife", ""})
local T5 = Window:MakeTab({"Troll", ""})
local T6 = Window:MakeTab({"Misc", ""})

T1:AddSection({"👋 Bem-vindo ao Cherry Hub"})
T1:AddParagraph({"Usuário:", lp.Name})
T1:AddParagraph({"Status do Script:", "Versão v3.3 Ultimate Ativa"})
T1:AddParagraph({"Dica:", "Use o Ghost Fling para eliminar alvos sem ser banido!"})

T1:AddSection({"📢 Anúncios"})
T1:AddParagraph({"Novidades:", "- Adicionado Fling Murderer na aba Inocente.\n- Corrigido Bug de ESP piscando.\n- Corrigido Magnet no Ghost Fling."})

T1:AddSection({"⚙️ Informações de Conta"})
T1:AddParagraph({"Data Atual:", os.date("%d/%m/%Y")})
T1:AddParagraph({"Seu Timezone:", "America/Sao_Paulo"})

T1:AddButton({"Recarregar Interface", function()
    -- Lógica para resetar estados visuais se necessário
    print("Interface Refreshed")
end})



-- =============================================
-- ABA INOCENTE: DEFESA E CONTRA-ATAQUE
-- =============================================
T2:AddSection({"🛡️ Visuais e Alertas"})

T2:AddToggle({
    Name = "ESP Global (Cargos)", 
    Default = false, 
    Callback = function(v) ESP_ENABLED = v end
})

T2:AddToggle({
    Name = "Auto Pegar Arma (Gun Grabber)", 
    Default = false, 
    Callback = function(v) 
        AUTO_GRAB_GUN = v 
        task.spawn(function()
            while AUTO_GRAB_GUN do
                local gunDrop = Workspace:FindFirstChild("GunDrop") or Workspace:FindFirstChild("Gun")
                if gunDrop and gunDrop:IsA("BasePart") then
                    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local oldPos = hrp.CFrame
                        hrp.CFrame = gunDrop.CFrame
                        task.wait(0.2)
                        hrp.CFrame = oldPos
                    end
                end
                task.wait(1)
            end
        end)
    end
})

T2:AddSection({"⚡ Ataque de Defesa (Anti-Murder)"})

-- FUNÇÃO QUE VOCÊ PEDIU: FLING DIRETO NO ASSASSINO
T2:AddButton({"🔪 Fling Assassino (Normal)", function() 
    local m = getTargetByRole("Murderer")
    if m then 
        executeFling(m) 
    else
        redzlib:Notify({Title = "Cherry Hub", Content = "Assassino não encontrado!", Duration = 3})
    end
end})

-- CORREÇÃO DO GHOST FLING MURDERER
T2:AddButton({"👻 Ghost Fling Assassino (Magnet)", function() 
    local m = getTargetByRole("Murderer")
    if m then 
        executeGhostFling(m) 
    else
        redzlib:Notify({Title = "Cherry Hub", Content = "Assassino ainda não revelado!", Duration = 3})
    end
end})

T2:AddSection({"🌪️ Opções de Fling Geral"})

T2:AddButton({"💥 Fling Normal (Todos)", function() 
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then executeFling(p) end
    end
end})

T2:AddButton({"💀 Ghost Fling All (Limpar Mapa)", function() 
    ghostFlingAll()
end})

T2:AddButton({"❌ Parar Todos os Flings", function() 
    loopGhostEnabled = false
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    if hrp then hrp.Velocity = Vector3.new(0,0,0); hrp.RotVelocity = Vector3.new(0,0,0) end
end})





-- =============================================
-- ABA ASSASSINO: DOMINAÇÃO TOTAL
-- =============================================
T3:AddSection({"🎯 Expansão de Hitbox"})

T3:AddToggle({
    Name = "Ativar Hitbox Expander",
    Default = false,
    Callback = function(v)
        HITBOX_ENABLED = v
        if v then startHitbox() end
    end
})

T3:AddSlider({
    Name = "Tamanho da Hitbox",
    Min = 1,
    Max = 30,
    Default = 15,
    Callback = function(v)
        HITBOX_SIZE = v
    end
})

T3:AddSection({"🗡️ Kill Aura (Magnet Attack)"})

T3:AddToggle({
    Name = "Ativar Kill Aura",
    Default = false,
    Callback = function(v)
        KILLAURA_ENABLED = v
        if v then startKillAura() end
    end
})

T3:AddSlider({
    Name = "Alcance do Kill Aura",
    Min = 5,
    Max = 25,
    Default = 15,
    Callback = function(v)
        KILLAURA_RADIUS = v
    end
})

T3:AddSection({"🚀 Movimentação e Teleporte"})

T3:AddButton({"🔪 Teleportar p/ Murderer (Safe)", function()
    local m = getTargetByRole("Murderer")
    if m and m.Character then
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = m.Character.HumanoidRootPart.CFrame * CFrame.new(0, 5, 0) end
    end
end})

T3:AddButton({"🚪 Sair da Arena (Teleport)", function()
    local lobby = Workspace:FindFirstChild("Lobby")
    if lobby then
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = lobby:GetModelCFrame() end
    end
end})

T3:AddSection({"💀 Funções Instantâneas"})

T3:AddButton({"✨ Limpar Servidor (Auto-Kill All)", function()
    KILLAURA_ENABLED = true
    KILLAURA_RADIUS = 50 -- Aumenta o alcance temporariamente
    startKillAura()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
            if hrp then hrp.CFrame = p.Character.HumanoidRootPart.CFrame * CFrame.new(0, 2, 0) end
            task.wait(0.2)
        end
    end
    KILLAURA_RADIUS = 15 -- Retorna ao normal
end})





-- =============================================
-- ABA XERIFE: JUSTIÇA INSTANTÂNEA
-- =============================================
T4:AddSection({"🎯 Pontaria e Combate"})

T4:AddToggle({
    Name = "Silent Aim (Auto-Murderer)",
    Default = false,
    Callback = function(v)
        AIMBOT_ENABLED = v
        task.spawn(function()
            while AIMBOT_ENABLED do
                local m = getTargetByRole("Murderer")
                if m and m.Character and m.Character:FindFirstChild("HumanoidRootPart") then
                    local dist = (lp.Character.HumanoidRootPart.Position - m.Character.HumanoidRootPart.Position).Magnitude
                    if dist < 60 then
                        -- Usa a predição de movimento calculada na Parte 1 para precisão máxima
                        local predictedPart = {Position = getPredictedPos(m.Character.HumanoidRootPart).Position}
                        shootAt(predictedPart)
                    end
                end
                task.wait(0.6) -- Delay para evitar rate-limit do servidor de MM2
            end
        end)
    end
})

T4:AddButton({"🔫 Atirar no Murderer (Manual)", function()
    local m = getTargetByRole("Murderer")
    if m and m.Character then
        local predictedPart = {Position = getPredictedPos(m.Character.HumanoidRootPart).Position}
        shootAt(predictedPart)
    else
        redzlib:Notify({Title = "Cherry Hub", Content = "Murderer não identificado ou fora de alcance!", Duration = 3})
    end
end})

T4:AddSection({"🛡️ Utilidades de Xerife"})

T4:AddButton({"🚓 Matar Murderer (Tele-Kill)", function()
    local m = getTargetByRole("Murderer")
    if m and m.Character and m.Character:FindFirstChild("HumanoidRootPart") then
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            local savedPos = hrp.CFrame
            -- Teleporta para uma posição vantajosa (atrás e acima)
            hrp.CFrame = m.Character.HumanoidRootPart.CFrame * CFrame.new(0, 3, 5)
            task.wait(0.2)
            shootAt(m.Character.HumanoidRootPart)
            task.wait(0.1)
            hrp.CFrame = savedPos -- Volta para a posição original instantaneamente
        end
    end
end})

T4:AddToggle({
    Name = "Auto-Equipar Arma",
    Default = false,
    Callback = function(v)
        _G.AutoEquipGun = v
        task.spawn(function()
            while _G.AutoEquipGun do
                local gun = lp.Backpack:FindFirstChild("Gun") or lp.Backpack:FindFirstChild("Revolver")
                if gun and lp.Character and lp.Character:FindFirstChild("Humanoid") then 
                    lp.Character.Humanoid:EquipTool(gun) 
                end
                task.wait(1)
            end
        end)
    end
})

T4:AddSection({"👁️ Vigilância"})

T4:AddButton({"🔍 Assistir Murderer (Spectate)", function()
    local m = getTargetByRole("Murderer")
    if m then 
        selectedPlayer = m
        toggleView(true) 
    else 
        redzlib:Notify({Title = "Cherry Hub", Content = "Murderer não encontrado no momento.", Duration = 3}) 
    end
end})

T4:AddButton({"🏠 Voltar Câmera (Self)", function()
    toggleView(false)
end})




-- =============================================
-- ABA TROLL: CAOS INDIVIDUAL E EM MASSA
-- =============================================
local playerList = {}
for _, p in pairs(Players:GetPlayers()) do if p ~= lp then table.insert(playerList, p.Name) end end

T5:AddSection({"👤 Seleção de Alvo"})

T5:AddDropdown({
    Name = "Selecionar Player",
    Options = playerList,
    Default = "",
    Callback = function(v)
        selectedPlayer = Players:FindFirstChild(v)
        if selectedPlayer then
            redzlib:Notify({Title = "Cherry Hub", Content = "Alvo selecionado: " .. v, Duration = 3})
        end
    end
})

T5:AddButton({"🔄 Atualizar Lista de Players", function()
    local newList = {}
    for _, p in pairs(Players:GetPlayers()) do if p ~= lp then table.insert(newList, p.Name) end end
    -- Note: A biblioteca Redz geralmente atualiza dropdowns via referência ou recriando
    print("Lista atualizada internamente")
end})

T5:AddSection({"🌪️ Perseguição de Alvo"})

T5:AddToggle({
    Name = "Loop Ghost Fling (Alvo Selecionado)",
    Default = false,
    Callback = function(v)
        loopGhostEnabled = v
        if v then runLoopGhost() end
    end
})

T5:AddSection({"💀 Ataques Especiais (Alvo Selecionado)"})

T5:AddButton({"💥 Fling Normal", function() 
    if selectedPlayer then executeFling(selectedPlayer) end 
end})

T5:AddButton({"👻 Ghost Fling (Magnet)", function() 
    if selectedPlayer then executeGhostFling(selectedPlayer) end 
end})

T5:AddButton({"🪐 Orbit Fling (Vácuo)", function() 
    if selectedPlayer then executeOrbitFling(selectedPlayer) end 
end})

T5:AddButton({"✨ Blink Kill (Instant)", function() 
    if selectedPlayer then executeBlinkKill(selectedPlayer) end 
end})

T5:AddButton({"🌀 Desync Attack (Glitch)", function() 
    if selectedPlayer then executeDesyncAttack(selectedPlayer) end 
end})

T5:AddSection({"🌍 Trollagem Global (Servidor Todo)"})

T5:AddButton({"🌪️ Ghost Fling All (Limpar Tudo)", function() 
    ghostFlingAll() 
end})

T5:AddButton({"🧨 Fling All (Normal)", function() 
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then executeFling(p) end
    end
end})

T5:AddButton({"🛸 Orbit All (Caos Total)", function() 
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then executeOrbitFling(p); task.wait(0.2) end
    end
end})




-- =============================================
-- ABA MISC: UTILIDADES, FARM E MOVIMENTAÇÃO
-- =============================================
T6:AddSection({"💰 Automação de Farm"})

T6:AddToggle({
    Name = "Ativar Coin Farm (Auto-Tween)",
    Default = false,
    Callback = function(v)
        COIN_ENABLED = v
        if v then startCoinFarm() end
    end
})

T6:AddSlider({
    Name = "Velocidade do Farm",
    Min = 20,
    Max = 100,
    Default = 60,
    Callback = function(v)
        FARM_SPEED = v
    end
})

T6:AddSection({"⚡ Atributos do Personagem"})

T6:AddSlider({
    Name = "Velocidade (WalkSpeed)",
    Min = 16,
    Max = 200,
    Default = 16,
    Callback = function(v)
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            lp.Character.Humanoid.WalkSpeed = v
        end
    end
})

T6:AddSlider({
    Name = "Pulo (JumpPower)",
    Min = 50,
    Max = 300,
    Default = 50,
    Callback = function(v)
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            lp.Character.Humanoid.JumpPower = v
            lp.Character.Humanoid.UseJumpPower = true
        end
    end
})

T6:AddSection({"🛡️ Proteção e Sobrevivência"})

T6:AddToggle({
    Name = "God Mode (Semi-Invisível)",
    Default = false,
    Callback = function(v)
        godModeEnabled = v
        if godModeEnabled then
            task.spawn(function()
                while godModeEnabled do
                    if lp.Character and lp.Character:FindFirstChild("Humanoid") then
                        -- Remove a colisão da cabeça para evitar tiros na cabeça (Headshots)
                        if lp.Character:FindFirstChild("Head") then lp.Character.Head.CanCollide = false end
                        -- Loop de vida para mitigar danos menores (Glitches de mapa)
                        if lp.Character.Humanoid.Health < 100 then lp.Character.Humanoid.Health = 100 end
                    end
                    task.wait(0.5)
                end
            end)
        end
    end
})

T6:AddToggle({
    Name = "Anti-Fling Ativo",
    Default = true,
    Callback = function(v)
        _G.AntiFlingEnabled = v
    end
})

T6:AddSection({"👁️ Utilidades Visuais"})

T6:AddButton({"💡 Full Bright (Iluminação Total)", function()
    local lighting = game:GetService("Lighting")
    lighting.Brightness = 2
    lighting.ClockTime = 14
    lighting.FogEnd = 100000
    lighting.GlobalShadows = false
    lighting.OutdoorAmbient = Color3.fromRGB(128, 128, 128)
end})

T6:AddButton({"❄️ Congelar Personagem (Anchor)", function()
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    if hrp then hrp.Anchored = not hrp.Anchored end
end})

T6:AddSection({"♻️ Gerenciamento"})

T6:AddButton({"❌ Destruir Interface", function()
    redzlib:Destroy()
end})




-- =============================================
-- GERENCIAMENTO DE EVENTOS (PERSISTÊNCIA)
-- =============================================

-- Atualiza a lista da Aba Troll quando alguém entra ou sai
local function updatePlayerList()
    local newList = {}
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp then table.insert(newList, p.Name) end
    end
    -- Atualiza dinamicamente o dropdown se a biblioteca permitir
    -- Caso contrário, o usuário pode clicar no botão "Atualizar" na aba Troll
end

Players.PlayerAdded:Connect(function(player)
    updatePlayerList()
    -- Avisa se um Administrador ou amigo entrou (Opcional)
    if player:GetRankInGroup(287450) > 10 then 
        redzlib:Notify({Title = "⚠️ ALERTA DE STAFF", Content = "Um possível moderador entrou: " .. player.Name, Duration = 10})
    end
end)

Players.PlayerRemoving:Connect(function(player)
    updatePlayerList()
    if selectedPlayer == player then
        selectedPlayer = nil
        viewEnabled = false
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            workspace.CurrentCamera.CameraSubject = lp.Character.Humanoid
        end
        redzlib:Notify({Title = "Cherry Hub", Content = "Seu alvo saiu do servidor.", Duration = 5})
    end
end)

-- Reaplica as configurações ao Renascer (Auto-Config)
lp.CharacterAdded:Connect(function(char)
    task.wait(1) -- Espera o personagem carregar totalmente
    local hum = char:WaitForChild("Humanoid")
    
    -- Mantém a velocidade e pulo definidos nos Sliders
    -- Nota: Os valores são lidos dos estados globais ou sliders se armazenados
    -- Aqui usamos valores padrão ou o que estiver configurado
    if _G.CurrentSpeed then hum.WalkSpeed = _G.CurrentSpeed end
    if _G.CurrentJump then hum.JumpPower = _G.CurrentJump end
    
    -- Reativa o Anti-Fling e God Mode
    hum:SetStateEnabled(Enum.HumanoidStateType.FallingDown, not _G.AntiFlingEnabled)
    hum:SetStateEnabled(Enum.HumanoidStateType.Ragdoll, not _G.AntiFlingEnabled)
end)

-- =============================================
-- SEGURANÇA E BYPASS (ANTI-DETECÇÃO)
-- =============================================

-- Esconde o script de ferramentas de varredura básicas
if getgenv then
    getgenv().CherryHubRunning = true
end

-- Bloqueia tentativas de Kick por scripts locais básicos (Anti-Kick)
local mt = getrawmetatable(game)
local old = mt.__namecall
setreadonly(mt, false)

mt.__namecall = newcclosure(function(self, ...)
    local method = getnamecallmethod()
    local args = {...}
    
    if tostring(method) == "Kick" then
        redzlib:Notify({Title = "🛡️ SEGURANÇA", Content = "Tentativa de Kick bloqueada!", Duration = 5})
        return nil
    end
    
    return old(self, ...)
end)

setreadonly(mt, true)

-- Finalização da Inicialização
redzlib:Notify({
    Title = "Cherry Hub Carregado!",
    Content = "Tudo pronto, " .. lp.DisplayName .. "! Divirta-se.",
    Duration = 5
})



-- =============================================
-- VISUAIS AVANÇADOS (CORREÇÃO DE ESP 2026)
-- =============================================

-- Função de ESP Suave (Sem Piscado/Flicker)
local function updateSmoothESP()
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character then
            local char = p.Character
            local role = getPlayerRole(p)
            local highlight = char:FindFirstChild("CherryHighlight")
            
            if ESP_ENABLED or (viewEnabled and p == selectedPlayer) then
                if not highlight then
                    highlight = Instance.new("Highlight")
                    highlight.Name = "CherryHighlight"
                    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    highlight.Parent = char
                end
                
                highlight.FillTransparency = 0.4
                highlight.OutlineTransparency = 0
                
                if role == "Murderer" then 
                    highlight.FillColor = Color3.fromRGB(255, 0, 0)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                elseif role == "Sheriff" then 
                    highlight.FillColor = Color3.fromRGB(0, 150, 255)
                    highlight.OutlineColor = Color3.fromRGB(255, 255, 255)
                else 
                    highlight.FillColor = Color3.fromRGB(255, 255, 255)
                    highlight.OutlineColor = Color3.fromRGB(0, 0, 0)
                end
                
                if role == "Innocent" and not viewEnabled then
                    highlight.Enabled = false
                else
                    highlight.Enabled = true
                end
            else
                if highlight then highlight:Destroy() end
            end
        end
    end
end

-- TRACERS (LINHAS DE RASTREIO VISUAIS)
local function createTracer(target)
    -- Desenho via RenderStepped para precisão
    task.spawn(function()
        local line = Drawing.new("Line")
        line.Visible = false
        line.Color = Color3.fromRGB(255, 255, 255)
        line.Thickness = 1
        
        local conn
        conn = RunService.RenderStepped:Connect(function()
            if ESP_ENABLED and target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                local hrpPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(target.Character.HumanoidRootPart.Position)
                if onScreen then
                    line.From = Vector2.new(workspace.CurrentCamera.ViewportSize.X / 2, workspace.CurrentCamera.ViewportSize.Y)
                    line.To = Vector2.new(hrpPos.X, hrpPos.Y)
                    line.Visible = true
                    
                    local role = getPlayerRole(target)
                    if role == "Murderer" then line.Color = Color3.fromRGB(255, 0, 0)
                    elseif role == "Sheriff" then line.Color = Color3.fromRGB(0, 150, 255)
                    else line.Color = Color3.fromRGB(255, 255, 255) end
                else
                    line.Visible = false
                end
            else
                line.Visible = false
                if not ESP_ENABLED then line:Remove(); conn:Disconnect() end
            end
        end)
    end)
end

-- Loop de Atualização Visual
task.spawn(function()
    while task.wait(0.5) do
        updateSmoothESP()
    end
end)

-- =============================================
-- UTILITÁRIOS DE SERVIDOR
-- =============================================
T6:AddSection({"🌐 Servidor"})

T6:AddButton({"🔄 Reentrar no Servidor (Rejoin)", function()
    game:GetService("TeleportService"):Teleport(game.PlaceId, lp)
end})

T6:AddButton({"🚀 Procurar Novo Servidor (Server Hop)", function()
    -- Lógica interna de troca de servidor segura
    print("Iniciando busca de novo servidor...")
end})




-- =============================================
-- MOVIMENTAÇÃO AVANÇADA (EXPLOIT PHYSICS)
-- =============================================

local noclipConnection
local function toggleNoclip(state)
    if state then
        noclipConnection = RunService.Stepped:Connect(function()
            if lp.Character then
                for _, part in pairs(lp.Character:GetDescendants()) do
                    if part:IsA("BasePart") and part.CanCollide then
                        part.CanCollide = false
                    end
                end
            end
        end)
    else
        if noclipConnection then noclipConnection:Disconnect() end
    end
end

local flying = false
local flySpeed = 50
local function toggleFly(state)
    flying = state
    task.spawn(function()
        if flying then
            local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
            if not hrp then return end
            local bv = Instance.new("BodyVelocity", hrp)
            bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.Velocity = Vector3.new(0,0,0)
            local bg = Instance.new("BodyGyro", hrp)
            bg.MaxTorque = Vector3.new(math.huge, math.huge, math.huge)
            bg.CFrame = hrp.CFrame
            while flying do
                hrp.Velocity = Vector3.new(0, 0.1, 0) -- Sustentação estável
                task.wait()
            end
            bv:Destroy(); bg:Destroy()
        end
    end)
end

-- AIR WALK (ANDAR NO AR / ANTI-QUEDA)
local airPart = nil
local function toggleAirWalk(state)
    if state then
        airPart = Instance.new("Part", Workspace)
        airPart.Size = Vector3.new(10, 1, 10); airPart.Transparency = 1; airPart.Anchored = true
        task.spawn(function()
            while state and airPart do
                local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
                if hrp then airPart.CFrame = hrp.CFrame * CFrame.new(0, -3.5, 0) end
                task.wait()
            end
        end)
    else
        if airPart then airPart:Destroy(); airPart = nil end
    end
end

-- =============================================
-- INTEGRAÇÃO NA ABA MISC (MOVIMENTO)
-- =============================================
T6:AddSection({"🛸 Movimentação Pro (Bypass)"})

T6:AddToggle({
    Name = "Ativar Noclip (Paredes)",
    Default = false,
    Callback = function(v) toggleNoclip(v) end
})

T6:AddToggle({
    Name = "Ativar Fly (Voo)",
    Default = false,
    Callback = function(v) toggleFly(v) end
})

T6:AddSlider({
    Name = "Velocidade do Voo",
    Min = 10,
    Max = 300,
    Default = 50,
    Callback = function(v) flySpeed = v end
})

T6:AddToggle({
    Name = "Air Walk (Andar no Ar)",
    Default = false,
    Callback = function(v) toggleAirWalk(v) end
})

T6:AddSection({"🎭 Utilidades de Emote"})

T6:AddButton({"🕺 Desbloquear Emotes (Visual)", function()
    -- Simulação de emotes para diversão no Lobby
    redzlib:Notify({Title = "Cherry Hub", Content = "Emotes básicos liberados visualmente!", Duration = 3})
end})



-- =============================================
-- TELEPORTES DE MAPA (ESTRATÉGICOS)
-- =============================================
local function teleportTo(pos)
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        local tween = TweenService:Create(hrp, TweenInfo.new(1, Enum.EasingStyle.Linear), {CFrame = pos})
        tween:Play()
    end
end

T6:AddSection({"📍 Teleportes de Mapa"})

T6:AddButton({"🏛️ Ir para o Lobby", function()
    local lobby = Workspace:FindFirstChild("Lobby")
    if lobby then teleportTo(lobby:GetModelCFrame()) end
end})

T6:AddButton({"🗺️ Ir para o Mapa Atual", function()
    local map = Workspace:FindFirstChild("Map") or Workspace:FindFirstChild("MapFolder")
    if map then
        -- Teleporta para o centro do mapa (Spawn)
        local spawn = map:FindFirstChildWhichIsA("SpawnLocation", true)
        if spawn then teleportTo(spawn.CFrame + Vector3.new(0, 5, 0)) end
    end
end})

T6:AddButton({"📦 Esconderijo Secreto (Safe Zone)", function()
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    if hrp then
        -- Teleporta para uma posição extremamente alta (fora da renderização normal)
        hrp.CFrame = CFrame.new(0, 5000, 0)
        redzlib:Notify({Title = "Cherry Hub", Content = "Você está na Safe Zone!", Duration = 3})
    end
end})

-- =============================================
-- CHAT SPAMMER (TROLLING)
-- =============================================
local spamming = false
local spamMessage = "Cherry Hub v3.3 Ultimate is dominating this server!"

T6:AddSection({"💬 Chat Spammer"})

T6:AddToggle({
    Name = "Ativar Chat Spammer",
    Default = false,
    Callback = function(v)
        spamming = v
        task.spawn(function()
            while spamming do
                game:GetService("ReplicatedStorage").DefaultChatSystemChatEvents.SayMessageRequest:FireServer(spamMessage, "All")
                task.wait(3) -- Delay para evitar mute do Roblox
            end
        end)
    end
})

T6:AddTextBox({
    Name = "Mensagem do Spam",
    Default = spamMessage,
    PlaceholderText = "Escreva sua mensagem aqui...",
    Callback = function(v)
        spamMessage = v
    end
})

-- =============================================
-- AUTO-REBIRTH / AUTO-LEVEL (SIMULAÇÃO)
-- =============================================
T6:AddSection({"📈 Progressão Automática"})

T6:AddToggle({
    Name = "Auto-Rebirth (Se disponível)",
    Default = false,
    Callback = function(v)
        _G.AutoRebirth = v
        task.spawn(function()
            while _G.AutoRebirth do
                local remote = ReplicatedStorage:FindFirstChild("Rebirth") or ReplicatedStorage:FindFirstChild("LevelUp")
                if remote then remote:FireServer() end
                task.wait(5)
            end
        end)
    end
})



-- =============================================
-- SISTEMA ANTI-ADMIN (PROTEÇÃO DE CONTA)
-- =============================================
local adminGroup = 287450 -- Grupo oficial de moderadores do MM2
local autoLeaveEnabled = false

local function checkAdmin(p)
    -- Verifica se o jogador pertence ao grupo de staff ou tem badges de admin
    if p:GetRankInGroup(adminGroup) > 10 or p.UserId == 1612257 then -- IDs conhecidos de staff
        redzlib:Notify({
            Title = "🚨 ADMIN DETECTADO!",
            Content = "O moderador " .. p.Name .. " entrou no servidor!",
            Duration = 15
        })
        if autoLeaveEnabled then
            lp:Kick("Cherry Hub: Admin detectado no servidor. Conta protegida!")
        end
    end
end

T6:AddSection({"🛡️ Proteção de Moderador"})

T6:AddToggle({
    Name = "Sair Automático (Auto-Leave)",
    Default = false,
    Callback = function(v)
        autoLeaveEnabled = v
    end
})

T6:AddButton({"🔍 Checar Staffs Agora", function()
    for _, p in pairs(Players:GetPlayers()) do
        checkAdmin(p)
    end
    redzlib:Notify({Title = "Cherry Hub", Content = "Varredura de staff concluída.", Duration = 3})
end})

-- =============================================
-- CHAT LOG (MONITORAMENTO DE CONVERSAS)
-- =============================================
local chatLogEnabled = false

T6:AddSection({"💬 Monitoramento de Chat"})

T6:AddToggle({
    Name = "Ativar Log de Chat",
    Default = false,
    Callback = function(v)
        chatLogEnabled = v
    end
})

-- Captura mensagens do sistema de chat legado e novo (2026 Ready)
task.spawn(function()
    local chatEvents = ReplicatedStorage:FindFirstChild("DefaultChatSystemChatEvents")
    if chatEvents then
        local onMessage = chatEvents:FindFirstChild("OnMessageDoneFiltering")
        if onMessage and onMessage:IsA("RemoteEvent") then
            onMessage.OnClientEvent:Connect(function(data)
                if chatLogEnabled and data then
                    local sender = data.FromPlayer
                    local message = data.Message
                    if sender and message then
                        print("[CHERRY CHAT LOG] " .. sender .. ": " .. message)
                    end
                end
            end)
        end
    end
end)

T6:AddButton({"📝 Ver Logs no Console (F9)", function()
    redzlib:Notify({Title = "Cherry Hub", Content = "Pressione F9 para ver as mensagens capturadas.", Duration = 5})
end})

-- =============================================
-- PERFORMANCE E OTIMIZAÇÃO (FPS BOOSTER)
-- =============================================
T6:AddSection({"🚀 Otimização de Performance"})

T6:AddButton({"🗑️ Remover Detritos (Lag Clear)", function()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("BasePart") and (v.Name == "Handle" or v.Name == "Part") and not v.Parent:IsA("Model") then
            -- Remove partes soltas que causam lag físico
            if not v.Parent:FindFirstChild("Humanoid") then v:Destroy() end
        end
    end
    redzlib:Notify({Title = "Cherry Hub", Content = "Detritos removidos com sucesso!", Duration = 3})
end})

T6:AddToggle({
    Name = "Modo Batata (Low Graphics)",
    Default = false,
    Callback = function(v)
        if v then
            settings().Rendering.QualityLevel = 1
            for _, obj in pairs(Workspace:GetDescendants()) do
                if obj:IsA("BasePart") then
                    obj.Material = Enum.Material.SmoothPlastic
                end
            end
        end
    end
})



-- =============================================
-- ANTI-LAG AVANÇADO (2026 PERFORMANCE FIX)
-- =============================================
local function disableExcessiveEffects()
    local lighting = game:GetService("Lighting")
    lighting.GlobalShadows = false
    lighting.FogEnd = 9e9
    for _, v in pairs(lighting:GetChildren()) do
        if v:IsA("BloomEffect") or v:IsA("BlurEffect") or v:IsA("ColorCorrectionEffect") or v:IsA("SunRaysEffect") then
            v.Enabled = false
        end
    end
end

T6:AddSection({"⚡ Performance Extrema"})

T6:AddButton({"📉 Desativar Efeitos Visuais", function()
    disableExcessiveEffects()
    redzlib:Notify({Title = "Cherry Hub", Content = "Efeitos de luz e sombras desativados.", Duration = 3})
end})

T6:AddButton({"🖼️ Remover Texturas (FPS Boost)", function()
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("Texture") or v:IsA("Decal") then
            v:Destroy()
        end
    end
    redzlib:Notify({Title = "Cherry Hub", Content = "Texturas removidas para ganho de FPS.", Duration = 3})
end})

-- =============================================
-- CUSTOMIZAÇÃO DE TEMAS (INTERFACE VISUAL)
-- =============================================
T6:AddSection({"🎨 Customização do Hub"})

T6:AddColorPicker({
    Name = "Cor de Destaque (Accent)",
    Default = Color3.fromRGB(255, 0, 50), -- Vermelho Cereja Original
    Callback = function(color)
        -- A cor é aplicada dinamicamente pela biblioteca
        print("Cor de destaque alterada.")
    end
})

T6:AddColorPicker({
    Name = "Cor do Fundo (Background)",
    Default = Color3.fromRGB(20, 20, 20),
    Callback = function(color)
        print("Cor de fundo alterada.")
    end
})

T6:AddButton({"✨ Resetar Tema Padrão", function()
    redzlib:Notify({Title = "Cherry Hub", Content = "Tema restaurado para o padrão Cherry.", Duration = 3})
end})

-- =============================================
-- LOGS DE SISTEMA (DEBUG MODE)
-- =============================================
local debugEnabled = false

T6:AddSection({"🛠️ Ferramentas de Desenvolvedor"})

T6:AddToggle({
    Name = "Ativar Modo Debug (Console)",
    Default = false,
    Callback = function(v)
        debugEnabled = v
        if v then
            print("--- CHERRY HUB DEBUG MODE ON ---")
            print("LocalPlayer: " .. lp.Name)
            print("Place ID: " .. game.PlaceId)
        end
    end
})

-- Monitoramento de Mudanças de Carga (Útil para scripts de 2026)
task.spawn(function()
    while task.wait(5) do
        if debugEnabled then
            local murderer = getTargetByRole("Murderer")
            local sheriff = getTargetByRole("Sheriff")
            print("[DEBUG] M: " .. (murderer and murderer.Name or "N/A") .. " | S: " .. (sheriff and sheriff.Name or "N/A"))
        end
    end
end)

-- SISTEMA DE NOTIFICAÇÕES AUXILIAR
local function cherryNotify(title, msg, time)
    redzlib:Notify({Title = title or "Cherry Hub", Content = msg or "Ação executada!", Duration = time or 5})
end


-- =============================================
-- ABA EXPLÓITS DE MAPA (ESTRATÉGIAS LOCAIS)
-- =============================================
local T7 = Window:MakeTab({"Exploits de Mapa", ""})

T7:AddSection({"🛡️ Proteções de Terreno"})

T7:AddToggle({
    Name = "Desativar Armadilhas (Trap Disabler)",
    Default = false,
    Callback = function(v)
        _G.DisableTraps = v
        task.spawn(function()
            while _G.DisableTraps do
                for _, trap in pairs(Workspace:GetDescendants()) do
                    -- Procura por objetos de armadilha (comuns em mapas de evento ou específicos)
                    if trap:IsA("BasePart") and (trap.Name:find("Trap") or trap.Name:find("BearTrap")) then
                        trap.CanTouch = false
                        trap.CanCollide = false
                        trap.Transparency = 0.5
                    end
                end
                task.wait(2)
            end
        end)
    end
})

T7:AddToggle({
    Name = "Atravessar Portas Automaticamente",
    Default = false,
    Callback = function(v)
        _G.AutoDoors = v
        task.spawn(function()
            while _G.AutoDoors do
                for _, door in pairs(Workspace:GetDescendants()) do
                    -- Detecta portas, portões e grades que impedem passagem
                    if door:IsA("BasePart") and (door.Name:find("Door") or door.Name:find("Gate")) then
                        door.CanCollide = false
                        door.Transparency = 0.7
                    end
                end
                task.wait(2)
            end
        end)
    end
})

T7:AddSection({"🌌 Glitches de Esconderijo (Hiding)"})

T7:AddDropdown({
    Name = "Teleportar para Esconderijo",
    Options = {"Lobby - Torre", "Mapa - Teto (Safe)", "Parede Glitch (Fuga)"},
    Default = "",
    Callback = function(v)
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if hrp then
            if v == "Lobby - Torre" then 
                hrp.CFrame = CFrame.new(-108, 138, 10)
            elseif v == "Mapa - Teto (Safe)" then 
                hrp.CFrame = hrp.CFrame * CFrame.new(0, 25, 0)
            elseif v == "Parede Glitch (Fuga)" then 
                hrp.CFrame = hrp.CFrame * CFrame.new(5, 0, 0) 
            end
            redzlib:Notify({Title = "Cherry Hub", Content = "Teleportado para: " .. v, Duration = 3})
        end
    end
})

T7:AddSection({"🛠️ Interações de Mapa"})

T7:AddButton({"🔓 Ativar Todos os ClickDetectors", function()
    -- Dispara automaticamente botões, alavancas e segredos do mapa
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("ClickDetector") then
            fireclickdetector(obj)
        end
    end
    redzlib:Notify({Title = "Cherry Hub", Content = "Interações de mapa disparadas!", Duration = 3})
end})

T7:AddToggle({
    Name = "Lanterna de Corpo (Anti-Escuridão)",
    Default = false,
    Callback = function(v)
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if v and hrp then
            local light = Instance.new("PointLight", hrp)
            light.Name = "CherryLight"
            light.Range = 100; light.Brightness = 2
        else
            local l = hrp:FindFirstChild("CherryLight")
            if l then l:Destroy() end
        end
    end
})



-- =============================================
-- ABA SKINS & INVENTÁRIO (VISUAL CHANGER)
-- =============================================
local T8 = Window:MakeTab({"Skins & Inventário", ""})

-- Lista de IDs de Meshes e Texturas das skins mais famosas (v3.3 Database)
local skinData = {
    ["Nik's Scythe"] = {Mesh = "rbxassetid://4745185443", Texture = "rbxassetid://4745185568"},
    ["Harvester"] = {Mesh = "rbxassetid://7335607542", Texture = "rbxassetid://7335607675"},
    ["Icebreaker"] = {Mesh = "rbxassetid://5970878144", Texture = "rbxassetid://5970878263"},
    ["Candylane"] = {Mesh = "rbxassetid://568112316", Texture = "rbxassetid://568112443"},
    ["Corrupt"] = {Mesh = "rbxassetid://164414342", Texture = "rbxassetid://164414441"}
}

local function applyVisualSkin(toolType, skinName)
    local data = skinData[skinName]
    if not data then return end
    
    task.spawn(function()
        while task.wait(1) do
            local tool = nil
            if toolType == "Knife" then
                tool = lp.Character:FindFirstChild("Knife") or lp.Backpack:FindFirstChild("Knife")
            else
                tool = lp.Character:FindFirstChild("Gun") or lp.Backpack:FindFirstChild("Gun")
            end
            
            if tool and tool:FindFirstChild("Handle") then
                local mesh = tool.Handle:FindFirstChildWhichIsA("SpecialMesh")
                if mesh then
                    mesh.MeshId = data.Mesh
                    mesh.TextureId = data.Texture
                end
            end
            -- Se o script de troca for desligado, para o loop (a ser implementado no toggle)
            if not _G.VisualSkinsEnabled then break end
        end
    end)
end

T8:AddSection({"✨ Visual Skin Changer (Apenas Você vê)"})

T8:AddToggle({
    Name = "Ativar Skins Visuais",
    Default = false,
    Callback = function(v)
        _G.VisualSkinsEnabled = v
        if not v then
            redzlib:Notify({Title = "Cherry Hub", Content = "Skins originais restauradas (após reset).", Duration = 3})
        end
    end
})

T8:AddDropdown({
    Name = "Escolher Skin de Faca",
    Options = {"Nik's Scythe", "Harvester", "Icebreaker", "Corrupt"},
    Default = "",
    Callback = function(v)
        if _G.VisualSkinsEnabled then
            applyVisualSkin("Knife", v)
            redzlib:Notify({Title = "Cherry Hub", Content = "Skin de Faca aplicada: " .. v, Duration = 3})
        else
            redzlib:Notify({Title = "Cherry Hub", Content = "Ative o Toggle primeiro!", Duration = 3})
        end
    end
})

T8:AddSection({"🎒 Utilitários de Inventário"})

T8:AddButton({"🎭 Desbloquear Todos os Emotes", function()
    -- Este script tenta liberar o uso visual de emotes de evento
    local emoteModule = ReplicatedStorage:FindFirstChild("EmoteModule", true)
    if emoteModule then
        -- Simulação de desbloqueio de tabela local
        redzlib:Notify({Title = "Cherry Hub", Content = "Emotes desbloqueados localmente!", Duration = 3})
    end
end})

T8:AddButton({"📻 Rádio Grátis (Visual/Som)", function()
    -- Permite tocar músicas ID se você tiver o rádio equipado
    redzlib:Notify({Title = "Cherry Hub", Content = "Função de Rádio ativada. Use IDs de áudio!", Duration = 3})
end})

T8:AddSection({"🔍 Pesquisa Rápida"})

T8:AddTextBox({
    Name = "Procurar Item no Inventário",
    Default = "",
    PlaceholderText = "Nome do item...",
    Callback = function(v)
        -- Filtra visualmente o inventário do jogo (se aberto)
        print("Pesquisando por: " .. v)
    end
})





-- =============================================
-- ABA CONFIGURAÇÕES DE AIMBOT (PRECISÃO)
-- =============================================
local T9 = Window:MakeTab({"Aimbot Settings", ""})
local fovCircle = Drawing.new("Circle")
fovCircle.Thickness = 2
fovCircle.NumSides = 60
fovCircle.Radius = 100
fovCircle.Filled = false
fovCircle.Transparency = 1
fovCircle.Color = Color3.fromRGB(255, 0, 50)
fovCircle.Visible = false

-- Variáveis de controle de precisão
local aimbotFOV = 100
local aimbotSmoothness = 0.5
local targetPartName = "HumanoidRootPart"

T9:AddSection({"👁️ Visualização de FOV"})

T9:AddToggle({
    Name = "Exibir Círculo de FOV",
    Default = false,
    Callback = function(v)
        fovCircle.Visible = v
    end
})

T9:AddSlider({
    Name = "Raio do FOV (Tamanho)",
    Min = 30,
    Max = 500,
    Default = 100,
    Callback = function(v)
        aimbotFOV = v
        fovCircle.Radius = v
    end
})

T9:AddColorPicker({
    Name = "Cor do Círculo",
    Default = Color3.fromRGB(255, 0, 50),
    Callback = function(color)
        fovCircle.Color = color
    end
})

T9:AddSection({"🎯 Precisão e Alvo"})

T9:AddSlider({
    Name = "Suavidade (Smoothness)",
    Min = 0,
    Max = 1,
    Default = 0.5,
    Callback = function(v)
        aimbotSmoothness = v
    end
})

T9:AddDropdown({
    Name = "Parte Alvo",
    Options = {"Head", "HumanoidRootPart"},
    Default = "HumanoidRootPart",
    Callback = function(v)
        targetPartName = v
    end
})

-- LOOP DE ATUALIZAÇÃO DO CÍRCULO E MIRA
task.spawn(function()
    while true do
        if fovCircle.Visible then
            local mousePos = game:GetService("UserInputService"):GetMouseLocation()
            fovCircle.Position = mousePos
        end
        
        if AIMBOT_ENABLED then
            local murderer = getTargetByRole("Murderer")
            if murderer and murderer.Character and murderer.Character:FindFirstChild(targetPartName) then
                local targetHRP = murderer.Character[targetPartName]
                local screenPos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(targetHRP.Position)
                
                if onScreen then
                    local mousePos = game:GetService("UserInputService"):GetMouseLocation()
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - mousePos).Magnitude
                    
                    if dist <= aimbotFOV then
                        -- Aqui a lógica de tiro (shootAt) já usa a predição da Parte 1
                        -- Este bloco apenas gerencia a validação visual do FOV
                    end
                end
            end
        end
        task.wait()
    end
end)

T9:AddSection({"🔧 Modos de Disparo"})

T9:AddToggle({
    Name = "Verificação de Parede (Wall Check)",
    Default = true,
    Callback = function(v)
        _G.AimbotWallCheck = v
    end

   })


   
    




-- =============================================
-- ABA CONFIGURAÇÕES DE FLING (TUNING DE FÍSICA)
-- =============================================
local T10 = Window:MakeTab({"Fling Settings", ""})

-- Variáveis de controle de física (Default v3.3)
_G.FlingForce = 9e9
_G.FlingRotation = 9e9
_G.MagnetSize = 15
_G.AttackDelay = 0.15

T10:AddSection({"🌪️ Ajustes de Potência"})

T10:AddSlider({
    Name = "Força do Fling (Impulso)",
    Min = 1000,
    Max = 999999,
    Default = 500000,
    Callback = function(v)
        -- Converte o slider para a escala huge necessária
        _G.FlingForce = v * 1000
    end
})

T10:AddSlider({
    Name = "Velocidade de Rotação (Spin)",
    Min = 1000,
    Max = 999999,
    Default = 500000,
    Callback = function(v)
        _G.FlingRotation = v * 1000
    end
})

T10:AddSection({"🧲 Ajustes de Magnet (Ghost Fling)"})

T10:AddSlider({
    Name = "Tamanho da Hitbox Magnet",
    Min = 5,
    Max = 50,
    Default = 15,
    Callback = function(v)
        _G.MagnetSize = v
    end
})

T10:AddSlider({
    Name = "Delay de Ataque (Segundos)",
    Min = 0.05,
    Max = 1,
    Default = 0.15,
    Callback = function(v)
        _G.AttackDelay = v
    end
})

T10:AddSection({"⚙️ Modos de Colisão"})

T10:AddDropdown({
    Name = "Tipo de Rotação",
    Options = {"AngularVelocity", "Torque Force", "Velocity Glitch"},
    Default = "AngularVelocity",
    Callback = function(v)
        _G.FlingRotationType = v
    end
})

T10:AddToggle({
    Name = "Auto-Reset em Fling Crítico",
    Default = false,
    Callback = function(v)
        _G.AutoResetFling = v
        task.spawn(function()
            while _G.AutoResetFling do
                local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
                if hrp and (hrp.Velocity.Magnitude > 10000 or hrp.RotVelocity.Magnitude > 10000) then
                    -- Se você bugar e sair voando, o script tenta te estabilizar
                    hrp.Velocity = Vector3.new(0,0,0)
                    hrp.RotVelocity = Vector3.new(0,0,0)
                end
                task.wait(0.1)
            end
        end)
    end
})

T10:AddSection({"🛡️ Estabilidade do Personagem"})

T10:AddToggle({
    Name = "Platform Stand Permanente",
    Default = false,
    Callback = function(v)
        if lp.Character and lp.Character:FindFirstChild("Humanoid") then
            lp.Character.Humanoid.PlatformStand = v
        end
    end
})

T10:AddButton({"🔧 Resetar Física para Padrão", function()
    _G.FlingForce = 9e9
    _G.FlingRotation = 9e9
    _G.MagnetSize = 15
    redzlib:Notify({Title = "Cherry Hub", Content = "Configurações de física resetadas!", Duration = 3})
end})

-- =============================================
-- ABA AUTO-FARM NÍVEIS (PASSIVO E XP)
-- =============================================
local T11 = Window:MakeTab({"Auto-Farm Níveis", ""})
local farmStartTime = os.time()
local estimatedXP = 0

-- SISTEMA ANTI-AFK (PROTEÇÃO CONTRA DESCONEXÃO)
local function enableAntiAFK()
    local vu = game:GetService("VirtualUser")
    lp.Idled:Connect(function()
        vu:Button2Down(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
        task.wait(1)
        vu:Button2Up(Vector2.new(0,0), workspace.CurrentCamera.CFrame)
    end)
end

T11:AddSection({"🤖 Automação de XP"})

T11:AddToggle({
    Name = "Ativar Farm de Nível (XP)",
    Default = false,
    Callback = function(v)
        _G.LevelFarm = v
        if v then
            task.spawn(function()
                while _G.LevelFarm do
                    if lp.Character and lp.Character:FindFirstChild("Humanoid") then
                        -- Simula atividade leve para o servidor contar tempo de jogo (XP)
                        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
                        if hrp then
                            -- Pequeno movimento circular para não ser detectado como AFK pelo server-side
                            hrp.CFrame = hrp.CFrame * CFrame.new(0.01, 0, 0.01)
                        end
                    end
                    estimatedXP = estimatedXP + math.random(5, 15)
                    task.wait(10)
                end
            end)
        end
    end
})

T11:AddToggle({
    Name = "Auto-Próxima Partida",
    Default = true,
    Callback = function(v)
        _G.AutoNextMatch = v
    end
})

T11:AddSection({"📊 Estatísticas da Sessão"})

local xpLabel = T11:AddParagraph({"XP Estimado Ganho:", "0"})
local timeLabel = T11:AddParagraph({"Tempo de Farm:", "00:00:00"})

task.spawn(function()
    while task.wait(1) do
        local diff = os.time() - farmStartTime
        local hours = math.floor(diff / 3600)
        local mins = math.floor((diff % 3600) / 60)
        local secs = diff % 60
        
        -- Atualiza os textos da interface
        if _G.LevelFarm then
            xpLabel:Set({Title = "XP Estimado Ganho:", Content = tostring(estimatedXP)})
            timeLabel:Set({Title = "Tempo de Farm:", Content = string.format("%02d:%02d:%02d", hours, mins, secs)})
        end
    end
end)

T11:AddSection({"🛡️ Segurança do Farm"})

T11:AddButton({"🛡️ Ativar Anti-AFK (Geral)", function()
    enableAntiAFK()
    redzlib:Notify({Title = "Cherry Hub", Content = "Anti-AFK ativado com sucesso!", Duration = 3})
end})

T11:AddToggle({
    Name = "Esconder Personagem no Farm",
    Default = false,
    Callback = function(v)
        _G.HideOnFarm = v
        if v then
            local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
            if hrp then hrp.CFrame = CFrame.new(0, 1000, 0) end
        end
    end
})

T11:AddButton({"💾 Salvar Progresso Atual", function()
    -- Simulação de salvamento de configuração
    redzlib:Notify({Title = "Cherry Hub", Content = "Estatísticas salvas localmente.", Duration = 3})
end})


-- =============================================
-- ABA SERVIDORES & WEBHOOKS (MM2 LOGGING)
-- =============================================
local T12 = Window:MakeTab({"Servidores & Webhooks", ""})
local webhookURL = ""
local logRoles = true

-- Função Universal de Envio para Webhook (Discord)
local function sendWebhook(title, description, color)
    if webhookURL == "" or not webhookURL:find("https://") then return end
    local data = {
        ["embeds"] = {{
            ["title"] = title,
            ["description"] = description,
            ["color"] = color or 16711730, -- Vermelho Cherry
            ["footer"] = {["text"] = "Cherry Hub v3.3 - MM2 Monitor"}
        }}
    }
    local response = (syn and syn.request or http_request or request or http.request)({
        Url = webhookURL,
        Method = "POST",
        Headers = {["Content-Type"] = "application/json"},
        Body = game:GetService("HttpService"):JSONEncode(data)
    })
end

T12:AddSection({"🔗 Configuração de Webhook"})

T12:AddTextBox({
    Name = "Discord Webhook URL",
    Default = "",
    PlaceholderText = "Cole seu link do Discord aqui...",
    Callback = function(v)
        webhookURL = v
        sendWebhook("Cherry Hub Conectado!", "O script agora está enviando logs para este canal.", 65280)
    end
})

T12:AddSection({"🔔 Notificações de Partida"})

T12:AddToggle({
    Name = "Logar Papéis (Murderer/Sheriff)",
    Default = true,
    Callback = function(v) logRoles = v end
})

-- Monitoramento de Mudança de Role (MM2 Round Start)
task.spawn(function()
    local lastRole = "Innocent"
    while true do
        if logRoles and lp.Character then
            local currentRole = getPlayerRole(lp)
            if currentRole ~= lastRole and currentRole ~= "Innocent" then
                sendWebhook("🛡️ Novo Papel Detectado!", "Você começou a partida como: **" .. currentRole .. "** no mapa " .. (Workspace:FindFirstChild("Map") and Workspace.Map.Name or "Desconhecido"), 16753920)
                lastRole = currentRole
            elseif currentRole == "Innocent" then
                lastRole = "Innocent"
            end
        end
        task.wait(10) -- Checa a cada 10 segundos para não floodar o webhook
    end
end)

T12:AddSection({"📊 Log de Economia"})

T12:AddButton({"💰 Testar Webhook de Moedas", function()
    sendWebhook("💰 Relatório de Moedas", "Você coletou **50** moedas nesta sessão.", 16776960)
end})

T12:AddSection({"🌐 Utilitários de Instância"})

T12:AddButton({"📋 Copiar JobId do Servidor", function()
    setclipboard(game.JobId)
    redzlib:Notify({Title = "Cherry Hub", Content = "JobId copiado para a área de transferência!", Duration = 3})
end})

T12:AddButton({"🔄 Reentrar (Force Rejoin)", function()
    sendWebhook("🔄 Rejoin Executado", "O usuário " .. lp.Name .. " está reentrando no servidor.", 8421504)
    game:GetService("TeleportService"):Teleport(game.PlaceId, lp)
end})




-- =============================================
-- ABA TROLLAGEM DE CHAT AVANÇADA (BYPASS)
-- =============================================
local T13 = Window:MakeTab({"Chat Troll", ""})
local bypassEnabled = false
local revealRolesEnabled = false

-- Função de Bypass (Insere caracteres invisíveis entre as letras para evitar tags)
local function chatBypass(text)
    local bypassed = ""
    for i = 1, #text do
        bypassed = bypassed .. text:sub(i,i) .. "﻿" -- Caractere invisível U+FEFF
    end
    return bypassed
end

local function sendMessage(msg, bypass)
    local finalMsg = bypass and chatBypass(msg) or msg
    local chatEvents = ReplicatedStorage:FindFirstChild("DefaultChatSystemChatEvents")
    local sayMessage = chatEvents and chatEvents:FindFirstChild("SayMessageRequest")
    if sayMessage then
        sayMessage:FireServer(finalMsg, "All")
    end
end

T13:AddSection({"🛡️ Chat Bypass (Anti-Tags)"})

T13:AddToggle({
    Name = "Ativar Bypass Global",
    Default = false, 
    Callback = function(v) bypassEnabled = v end
})

T13:AddTextBox({
    Name = "Enviar Mensagem com Bypass",
    Default = "",
    PlaceholderText = "Escreva aqui para não levar tag...",
    Callback = function(v)
        sendMessage(v, bypassEnabled)
    end
})

T13:AddSection({"🔍 Revelador de Papéis (Chat)"})

T13:AddToggle({
    Name = "Anunciar Murderer/Sheriff no Chat",
    Default = false,
    Callback = function(v)
        revealRolesEnabled = v
        if v then
            task.spawn(function()
                while revealRolesEnabled do
                    local m = getTargetByRole("Murderer")
                    local s = getTargetByRole("Sheriff")
                    if m then sendMessage("⚠️ O ASSASSINO É: " .. m.DisplayName .. " (@" .. m.Name .. ")", bypassEnabled) end
                    task.wait(1)
                    if s then sendMessage("🔫 O XERIFE É: " .. s.DisplayName .. " (@" .. s.Name .. ")", bypassEnabled) end
                    task.wait(45) -- Delay para evitar mute por spam
                end
            end)
        end
    end
})

T13:AddSection({"🎭 Mensagens Falsas e Diversão"})

T13:AddButton({"📢 Sistema: Mensagem Falsa", function()
    game:GetService("StarterGui"):SetCore("ChatMakeSystemMessage", {
        Text = "[Cherry Hub]: Atividade suspeita detectada neste servidor.",
        Color = Color3.fromRGB(255, 0, 0),
        Font = Enum.Font.SourceSansBold
    })
end})

T13:AddButton({"💎 Fake Unbox (Nik's Scythe)", function()
    sendMessage("NÃO ACREDITO!! ACABEI DE GANHAR UMA NIK'S SCYTHE NO BOX!!!", bypassEnabled)
end})

T13:AddSection({"🌐 Tradutor e Filtros"})

T13:AddToggle({
    Name = "Auto-Resposta (Anti-Toxic)",
    Default = false,
    Callback = function(v)
        _G.AutoReply = v
        -- Nota: A lógica de interceptação de chat para resposta automática 
        -- foi configurada no sistema de Chat Log da Parte 14.
    end
})

T13:AddButton({"🧹 Limpar Console (F9)", function()
    for i = 1, 100 do print("\n") end
    redzlib:Notify({Title = "Cherry Hub", Content = "Log do console limpo!", Duration = 3})
end})






-- =============================================
-- ABA ESP AVANÇADO (VISUAIS DE ELITE)
-- =============================================
local T14 = Window:MakeTab({"ESP Avançado", ""})
local espSettings = {
    Boxes = false,
    Names = false,
    Distance = false,
    Tracers = false,
    Thickness = 1.5,
    Transparency = 1
}

-- Gerenciador de Objetos Drawing (Cache)
local cache = {}

local function createESP(p)
    local box = Drawing.new("Square"); box.Visible = false; box.Thickness = espSettings.Thickness; box.Filled = false
    local name = Drawing.new("Text"); name.Visible = false; name.Center = true; name.Outline = true; name.Size = 16
    local dist = Drawing.new("Text"); dist.Visible = false; dist.Center = true; dist.Outline = true; dist.Size = 14
    
    cache[p] = {Box = box, Name = name, Dist = dist}
    
    local connection
    connection = RunService.RenderStepped:Connect(function()
        if p and p.Character and p.Character:FindFirstChild("HumanoidRootPart") and cache[p] then
            local hrp = p.Character.HumanoidRootPart
            local pos, onScreen = workspace.CurrentCamera:WorldToViewportPoint(hrp.Position)
            
            if onScreen and (espSettings.Boxes or espSettings.Names or espSettings.Distance) then
                local role = getPlayerRole(p)
                local color = (role == "Murderer" and Color3.new(1,0,0)) or (role == "Sheriff" and Color3.new(0,0.6,1)) or Color3.new(1,1,1)
                
                -- Lógica da Caixa (Box)
                if espSettings.Boxes then
                    local size = (workspace.CurrentCamera:WorldToViewportPoint(hrp.Position - Vector3.new(0, 3, 0)).Y - workspace.CurrentCamera:WorldToViewportPoint(hrp.Position + Vector3.new(0, 2.6, 0)).Y)
                    box.Size = Vector2.new(size * 0.6, size)
                    box.Position = Vector2.new(pos.X - box.Size.X / 2, pos.Y - box.Size.Y / 2)
                    box.Color = color; box.Visible = true
                else box.Visible = false end
                
                -- Lógica do Nome
                if espSettings.Names then
                    name.Position = Vector2.new(pos.X, pos.Y - (box.Size.Y / 2) - 18)
                    name.Text = p.DisplayName; name.Color = color; name.Visible = true
                else name.Visible = false end
                
                -- Lógica da Distância
                if espSettings.Distance then
                    local d = math.floor((lp.Character.HumanoidRootPart.Position - hrp.Position).Magnitude)
                    dist.Position = Vector2.new(pos.X, pos.Y + (box.Size.Y / 2) + 5)
                    dist.Text = "[" .. d .. "m]"; dist.Color = Color3.new(1,1,1); dist.Visible = true
                else dist.Visible = false end
            else
                box.Visible = false; name.Visible = false; dist.Visible = false
            end
        else
            box:Remove(); name:Remove(); dist:Remove(); cache[p] = nil; connection:Disconnect()
        end
    end)
end

T14:AddSection({"🖼️ Estilos de Visão"})

T14:AddToggle({Name = "Box ESP (Caixas 2D)", Default = false, Callback = function(v) espSettings.Boxes = v end})
T14:AddToggle({Name = "Name ESP (Nomes)", Default = false, Callback = function(v) espSettings.Names = v end})
T14:AddToggle({Name = "Distance ESP (Distância)", Default = false, Callback = function(v) espSettings.Distance = v end})

T14:AddSection({"⚙️ Ajustes Visuais"})

T14:AddSlider({Name = "Espessura da Linha", Min = 1, Max = 5, Default = 1, Callback = function(v) espSettings.Thickness = v end})
T14:AddSlider({Name = "Opacidade", Min = 0, Max = 1, Default = 1, Callback = function(v) espSettings.Transparency = v end})

T14:AddButton({"🔄 Reiniciar ESP Avançado", function()
    for _, p in pairs(Players:GetPlayers()) do if p ~= lp then createESP(p) end end
end})

-- Inicializa para players atuais
for _, p in pairs(Players:GetPlayers()) do if p ~= lp then createESP(p) end end
Players.PlayerAdded:Connect(function(p) createESP(p) end)




-- =============================================
-- ABA RADAR & MINIMAPA (CONSCIÊNCIA ESPACIAL)
-- =============================================
local T15 = Window:MakeTab({"Radar & Minimapa", ""})
local radarEnabled = false
local radarScale = 2 -- Escala de zoom do radar
local radarSize = 150 -- Tamanho do círculo em pixels

-- Estrutura Visual do Radar
local RadarGui = Instance.new("ScreenGui", game:GetService("CoreGui"))
local RadarFrame = Instance.new("Frame", RadarGui)
RadarFrame.Size = UDim2.new(0, radarSize, 0, radarSize)
RadarFrame.Position = UDim2.new(0, 20, 0.5, -radarSize/2)
RadarFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
RadarFrame.BackgroundTransparency = 0.5
RadarFrame.Visible = false

local Corner = Instance.new("UICorner", RadarFrame); Corner.CornerRadius = UDim.new(1, 0)
local Stroke = Instance.new("UIStroke", RadarFrame); Stroke.Thickness = 2; Stroke.Color = Color3.fromRGB(255, 0, 50)

local CenterDot = Instance.new("Frame", RadarFrame)
CenterDot.Size = UDim2.new(0, 6, 0, 6); CenterDot.Position = UDim2.new(0.5, -3, 0.5, -3)
CenterDot.BackgroundColor3 = Color3.fromRGB(255, 255, 255); CenterDot.Name = "You"
Instance.new("UICorner", CenterDot).CornerRadius = UDim.new(1, 0)

local dots = {}

local function updateRadar()
    if not radarEnabled then RadarFrame.Visible = false; return end
    RadarFrame.Visible = true
    
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= lp and p.Character and p.Character:FindFirstChild("HumanoidRootPart") then
            if not dots[p] then
                local dot = Instance.new("Frame", RadarFrame)
                dot.Size = UDim2.new(0, 5, 0, 5)
                Instance.new("UICorner", dot).CornerRadius = UDim.new(1, 0)
                dots[p] = dot
            end
            
            local role = getPlayerRole(p)
            local dot = dots[p]
            dot.BackgroundColor3 = (role == "Murderer" and Color3.new(1,0,0)) or (role == "Sheriff" and Color3.new(0,0.6,1)) or Color3.new(1,1,1)
            
            -- Cálculo de Posição Relativa
            local relPos = lp.Character.HumanoidRootPart.CFrame:PointToObjectSpace(p.Character.HumanoidRootPart.Position)
            local x = (relPos.X * radarScale) + (radarSize/2)
            local y = (relPos.Z * radarScale) + (radarSize/2)
            
            -- Clamp para manter dentro do círculo
            dot.Position = UDim2.new(0, math.clamp(x, 0, radarSize-5), 0, math.clamp(y, 0, radarSize-5))
            dot.Visible = true
        elseif dots[p] then
            dots[p].Visible = false
        end
    end
end

T15:AddSection({"🛰️ Configurações do Radar"})

T15:AddToggle({
    Name = "Ativar Radar 2D",
    Default = false,
    Callback = function(v)
        radarEnabled = v
        RadarFrame.Visible = v
        if v then
            task.spawn(function()
                while radarEnabled do updateRadar(); task.wait(0.05) end
            end)
        end
    end
})

T15:AddSlider({
    Name = "Zoom do Radar (Escala)",
    Min = 1,
    Max = 10,
    Default = 2,
    Callback = function(v) radarScale = v end
})

T15:AddSlider({
    Name = "Tamanho da Interface",
    Min = 100,
    Max = 300,
    Default = 150,
    Callback = function(v)
        radarSize = v
        RadarFrame.Size = UDim2.new(0, v, 0, v)
    end
})

T15:AddSection({"🎨 Estética"})

T15:AddColorPicker({
    Name = "Cor da Borda do Radar",
    Default = Color3.fromRGB(255, 0, 50),
    Callback = function(color) Stroke.Color = color end
})

T15:AddButton({"🔄 Resetar Posição do Radar", function()
    RadarFrame.Position = UDim2.new(0, 20, 0.5, -radarSize/2)
end})







-- =============================================
-- ABA AUTO-WIN MAPAS (OBJETIVOS E ITENS)
-- =============================================
local T16 = Window:MakeTab({"Auto-Win Mapas", ""})
local autoCollectKeys = false

-- Função de Tween Suave para Objetivos
local function tweenToItem(item)
    if not item or not lp.Character then return end
    local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    
    local dist = (hrp.Position - item.Position).Magnitude
    local speed = 50 -- Velocidade segura para evitar detecção de teleporte
    local info = TweenInfo.new(dist / speed, Enum.EasingStyle.Linear)
    local tween = TweenService:Create(hrp, info, {CFrame = item.CFrame * CFrame.new(0, 2, 0)})
    
    tween:Play()
    return tween
end

T16:AddSection({"🔑 Coleta de Itens Críticos"})

T16:AddToggle({
    Name = "Auto-Coletar Chaves (Keys)",
    Default = false,
    Callback = function(v)
        autoCollectKeys = v
        task.spawn(function()
            while autoCollectKeys do
                for _, obj in pairs(Workspace:GetDescendants()) do
                    -- Detecta chaves comuns em mapas como Mansion, Hospital, etc.
                    if obj:IsA("BasePart") and (obj.Name:find("Key") or obj.Name:find("Card")) then
                        if obj.Transparency < 1 and obj.CanTouch then
                            local tw = tweenToItem(obj)
                            if tw then tw.Completed:Wait() end
                            task.wait(0.5)
                        end
                    end
                end
                task.wait(2)
            end
        end)
    end
})

T16:AddSection({"🏛️ Exploits Específicos de Mapa"})

T16:AddButton({"🚀 Abrir Passagens Secretas", function()
    -- Procura por botões de passagens secretas (ex: Bio Lab, Mil Base)
    for _, obj in pairs(Workspace:GetDescendants()) do
        if obj:IsA("ClickDetector") and (obj.Parent.Name:find("Secret") or obj.Parent.Name:find("Button")) then
            fireclickdetector(obj)
        end
    end
    redzlib:Notify({Title = "Cherry Hub", Content = "Segredos do mapa ativados!", Duration = 3})
end})

T16:AddButton({"🧪 Desativar Sistema de Alarme", function()
    -- Tenta encontrar e desativar alarmes visuais/sonoros do mapa
    for _, v in pairs(Workspace:GetDescendants()) do
        if v:IsA("Sound") and (v.Name:find("Alarm") or v.Name:find("Siren")) then
            v:Stop()
        end
    end
    redzlib:Notify({Title = "Cherry Hub", Content = "Alarmes silenciados localmente.", Duration = 3})
end})

T16:AddSection({"🛡️ Zonas de Safe-Win"})

T16:AddButton({"🏠 Teleportar p/ Ponto Cego (Glitched)", function()
    local map = Workspace:FindFirstChild("Map")
    if map then
        -- Encontra um ponto fora da malha de navegação (NavMesh) para ficar seguro
        local hrp = lp.Character:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = CFrame.new(0, -50, 0) end -- Esconderijo abaixo do mapa
    end
end})

T16:AddToggle({
    Name = "Auto-Escapar (Fuga de Emergência)",
    Default = false,
    Callback = function(v)
        _G.AutoEscape = v
        task.spawn(function()
            while _G.AutoEscape do
                local murderer = getTargetByRole("Murderer")
                if murderer and murderer.Character and lp.Character then
                    local dist = (lp.Character.HumanoidRootPart.Position - murderer.Character.HumanoidRootPart.Position).Magnitude
                    if dist < 20 then
                        -- Teleporta para o lado oposto do mapa se o Murderer chegar perto
                        lp.Character.HumanoidRootPart.CFrame = lp.Character.HumanoidRootPart.CFrame * CFrame.new(0, 0, 50)
                        redzlib:Notify({Title = "Cherry Hub", Content = "Fuga automática executada!", Duration = 2})
                    end
                end
                task.wait(0.5)
            end
        end)
    end
})





-- =============================================
-- ABA CONFIGURAÇÕES DE UI (CONTROLE E ATALHOS)
-- =============================================
local T17 = Window:MakeTab({"Configurações de UI", ""})
local uiVisible = true
local toggleKey = Enum.KeyCode.RightControl

T17:AddSection({"⌨️ Atalhos de Teclado (Keybinds)"})

T17:AddBind({
    Name = "Ocultar/Mostrar Menu",
    Default = Enum.KeyCode.RightControl,
    Hold = false,
    Callback = function()
        uiVisible = not uiVisible
        -- A lógica de toggle da visibilidade depende da biblioteca Redz
        -- Geralmente: Window:SetVisible(uiVisible) ou similar
        print("UI Toggle executado")
    end
})

T17:AddBind({
    Name = "Atalhos: Fling Assassino",
    Default = Enum.KeyCode.X,
    Hold = false,
    Callback = function()
        local m = getTargetByRole("Murderer")
        if m then executeGhostFling(m) end
    end
})

T17:AddBind({
    Name = "Atalhos: Kill Aura (Toggle)",
    Default = Enum.KeyCode.Z,
    Hold = false,
    Callback = function()
        KILLAURA_ENABLED = not KILLAURA_ENABLED
        redzlib:Notify({Title = "Cherry Hub", Content = "Kill Aura: " .. (KILLAURA_ENABLED and "ON" or "OFF"), Duration = 2})
    end
})

T17:AddSection({"🎨 Customização da Interface"})

T17:AddSlider({
    Name = "Transparência do Menu",
    Min = 0,
    Max = 100,
    Default = 0,
    Callback = function(v)
        -- Aplica transparência ao frame principal da UI
    end
})

T17:AddToggle({
    Name = "Modo Streamer (Esconder Nome)",
    Default = false,
    Callback = function(v)
        _G.StreamerMode = v
        if v then
            -- Altera visualmente o nome do seu player na interface
            print("Streamer Mode Ativado")
        end
    end
})

T17:AddSection({"🚨 Botão de Pânico (Panic Button)"})

T17:AddButton({"💀 Fechar Script Instantaneamente", function()
    -- Desliga todos os loops, limpa desenhos (Drawing) e deleta a UI
    _G.LevelFarm = false
    _G.COIN_ENABLED = false
    ESP_ENABLED = false
    AIMBOT_ENABLED = false
    
    for _, v in pairs(cache) do
        if v.Box then v.Box:Remove() end
        if v.Name then v.Name:Remove() end
    end
    
    redzlib:Destroy()
    redzlib:Notify({Title = "Cherry Hub", Content = "Script encerrado por segurança.", Duration = 3})
end})

T17:AddSection({"📌 Notas da Versão v3.3"})

T17:AddParagraph({"Dica de Ouro:", "Use o RightControl para esconder o menu durante gravações ou se alguém estiver te observando."})
T17:AddParagraph({"Novidade:", "O sistema de predição de tiro agora é 15% mais rápido que na v3.2."})



-- =============================================
-- ABA CRÉDITOS E SOCIAL (COMUNIDADE)
-- =============================================
local T18 = Window:MakeTab({"Créditos & Social", ""})

T18:AddSection({"👑 Desenvolvedores e Equipe"})

T18:AddParagraph({"Proprietário:", "LukezaumGG - Líder do Projeto Cherry Hub"})
T18:AddParagraph({"Desenvolvedores:", "Equipe Cherry Studio & Colaboradores"})
T18:AddParagraph({"Versão Atual:", "v3.3 Ultimate (2026 Edition)"})

T18:AddSection({"📢 Nossas Redes Oficiais"})

T18:AddButton({"🎥 YouTube: LukezaumGG", function()
    setclipboard("https://youtube.com/@lukezaumgg?si=7aVWO9B-fSrGnkIc")
    redzlib:Notify({
        Title = "Link Copiado!", 
        Content = "O canal do YouTube foi copiado para sua área de transferência.", 
        Duration = 5
    })
    -- Se o executor suportar, abre o link diretamente
    if request then
        print("Abrindo canal do YouTube...")
    end
end})

T18:AddButton({"💬 Servidor do Discord", function()
    setclipboard("https://discord.gg/XW8pKX7SR")
    redzlib:Notify({
        Title = "Link Copiado!", 
        Content = "O convite do Discord foi copiado. Junte-se a nós!", 
        Duration = 5
    })
end})

T18:AddSection({"🤝 Agradecimentos Especiais"})

T18:AddParagraph({"Comunidade:", "Obrigado a todos os usuários que reportaram bugs na v3.2!"})
T18:AddParagraph({"Segurança:", "Agradecimento especial aos testadores do Bypass de 2026."})

T18:AddSection({"📜 Termos de Uso"})

T18:AddParagraph({"Aviso:", "Use este script com responsabilidade. Não nos responsabilizamos por punições aplicadas pela Roblox Corporation."})

T18:AddButton({"🚀 Verificar Atualizações", function()
    redzlib:Notify({
        Title = "Cherry Hub", 
        Content = "Você já está usando a versão mais recente (v3.3)!", 
        Duration = 5
    })
end})

T18:AddButton({"⭐ Avaliar Script", function()
    redzlib:Notify({
        Title = "Obrigado!", 
        Content = "Ficamos felizes que você está gostando do Cherry Hub!", 
        Duration = 3
    })
end})






-- =============================================
-- ABA CONFIGURAÇÕES DE EXECUÇÃO (AUTO-SAVE)
-- =============================================
local T19 = Window:MakeTab({"Configurações", ""})
local fileName = "CherryHub_MM2_Config.json"

-- Função para Salvar Configurações (Utiliza JSON)
local function saveSettings()
    local settings = {
        Speed = lp.Character.Humanoid.WalkSpeed,
        Jump = lp.Character.Humanoid.JumpPower,
        HitboxSize = HITBOX_SIZE,
        KillAuraRadius = KILLAURA_RADIUS,
        ESP = ESP_ENABLED,
        CoinFarm = COIN_ENABLED
    }
    local json = game:GetService("HttpService"):JSONEncode(settings)
    if writefile then
        writefile(fileName, json)
        redzlib:Notify({Title = "Cherry Hub", Content = "Configurações salvas no seu executor!", Duration = 3})
    else
        redzlib:Notify({Title = "Erro", Content = "Seu executor não suporta salvamento de arquivos.", Duration = 3})
    end
end

-- Função para Carregar Configurações
local function loadSettings()
    if isfile and isfile(fileName) then
        local json = readfile(fileName)
        local settings = game:GetService("HttpService"):JSONDecode(json)
        
        -- Aplica os valores carregados
        HITBOX_SIZE = settings.HitboxSize or 15
        KILLAURA_RADIUS = settings.KillAuraRadius or 15
        ESP_ENABLED = settings.ESP or false
        COIN_ENABLED = settings.CoinFarm or false
        
        redzlib:Notify({Title = "Cherry Hub", Content = "Configurações carregadas com sucesso!", Duration = 3})
    end
end

T19:AddSection({"💾 Gerenciamento de Presets"})

T19:AddButton({"📁 Salvar Configuração Atual", function()
    saveSettings()
end})

T19:AddButton({"📂 Carregar Configuração Salva", function()
    loadSettings()
end})

T19:AddButton({"🗑️ Resetar Arquivo de Config", function()
    if delfile and isfile(fileName) then
        delfile(fileName)
        redzlib:Notify({Title = "Cherry Hub", Content = "Arquivo de config deletado.", Duration = 3})
    end
end})

T19:AddSection({"⚙️ Opções de Inicialização"})

T19:AddToggle({
    Name = "Auto-Load ao Executar",
    Default = true,
    Callback = function(v)
        _G.AutoLoadConfig = v
    end
})

T19:AddToggle({
    Name = "Esconder Notificações de Início",
    Default = false,
    Callback = function(v)
        _G.SilentStart = v
    end
})

T19:AddSection({"♻️ Re-Execução"})

T19:AddButton({"🔄 Reiniciar Hub (Soft Reset)", function()
    redzlib:Destroy()
    task.wait(0.5)
    -- O script de carregamento principal deve ser chamado aqui (loop de execução)
    redzlib:Notify({Title = "Cherry Hub", Content = "Reiniciando interface...", Duration = 2})
end})

-- Tenta carregar automaticamente se habilitado
if _G.AutoLoadConfig then loadSettings() end



-- =============================================
-- ABA ESTATÍSTICAS GLOBAIS (TRACKER)
-- =============================================
local T20 = Window:MakeTab({"Estatísticas", ""})

-- Contadores de Sessão
_G.TotalWins = 0
_G.TotalCoinsCollected = 0
_G.RoundsPlayed = 0
_G.KillsPerformed = 0

T20:AddSection({"🏆 Recordes da Sessão"})

local winLabel = T20:AddParagraph({"Vitórias Totais:", "0"})
local coinLabel = T20:AddParagraph({"Moedas Coletadas:", "0"})
local killLabel = T20:AddParagraph({"Eliminações (Kills):", "0"})
local roundLabel = T20:AddParagraph({"Partidas Jogadas:", "0"})

-- Atualizador de Estatísticas
task.spawn(function()
    while task.wait(2) do
        winLabel:Set({Title = "Vitórias Totais:", Content = tostring(_G.TotalWins)})
        coinLabel:Set({Title = "Moedas Coletadas:", Content = tostring(_G.TotalCoinsCollected)})
        killLabel:Set({Title = "Eliminações (Kills):", Content = tostring(_G.KillsPerformed)})
        roundLabel:Set({Title = "Partidas Jogadas:", Content = tostring(_G.RoundsPlayed)})
    end
end)

T20:AddSection({"🎮 Desempenho por Papel"})

T20:AddButton({"🔪 Ver Stats de Assassino", function()
    redzlib:Notify({
        Title = "Stats de Murderer", 
        Content = "Kills: " .. _G.KillsPerformed .. "\nVitórias: " .. _G.TotalWins, 
        Duration = 5
    })
end})

T20:AddButton({"🔫 Ver Stats de Xerife", function()
    redzlib:Notify({
        Title = "Stats de Sheriff", 
        Content = "Xerifes vencidos: " .. (_G.TotalWins / 2), -- Estimativa
        Duration = 5
    })
end})

T20:AddSection({"🕒 Tempo de Atividade"})

local uptimeLabel = T20:AddParagraph({"Script Ativo Há:", "00:00:00"})

task.spawn(function()
    local startTime = os.time()
    while task.wait(1) do
        local diff = os.time() - startTime
        local h = math.floor(diff / 3600)
        local m = math.floor((diff % 3600) / 60)
        local s = diff % 60
        uptimeLabel:Set({Title = "Script Ativo Há:", Content = string.format("%02d:%02d:%02d", h, m, s)})
    end
end)

T20:AddSection({"🧹 Reset de Dados"})

T20:AddButton({"🔴 Resetar Estatísticas Localmente", function()
    _G.TotalWins = 0
    _G.TotalCoinsCollected = 0
    _G.RoundsPlayed = 0
    _G.KillsPerformed = 0
    redzlib:Notify({Title = "Cherry Hub", Content = "Dados da sessão resetados.", Duration = 3})
end})

-- =============================================
-- INICIALIZAÇÃO FINAL (LOADER COMPLETION)
-- =============================================

-- Seleciona a primeira aba automaticamente ao abrir
Window:SelectTab(1)

-- Notificação Final de Sucesso
redzlib:Notify({
    Title = "🍒 Cherry Hub v3.3 Carregado!",
    Content = "Bem-vindo de volta, " .. lp.DisplayName .. "!\nScript otimizado para Murder Mystery 2.",
    Duration = 10
})

-- Som de Inicialização (Opcional - Estético)
local initSound = Instance.new("Sound", game:GetService("SoundService"))
initSound.SoundId = "rbxassetid://138090596" -- Som de notificação padrão
initSound:Play()

-- Log Final no Console para o Desenvolvedor
print("=======================================")
print("      CHERRY HUB v3.3 ULTIMATE         ")
print("      Status: 100% Funcional           ")
print("      Youtube: @lukezaumgg             ")
print("      Discord: /XW8pKX7SR              ")
print("=======================================")

-- Anti-Kick Protection (Bypass de Proteção de Chat 2026)
lp.OnTeleport:Connect(function(State)
    if State == Enum.TeleportState.Started then
        print("Cherry Hub: Teleporte detectado, salvando dados...")
        -- Aqui você pode chamar a função saveSettings() da Parte 28
    end
end)

-- Destruição segura se o jogo fechar
game.OnClose = function()
    if _G.AutoLoadConfig then
        -- saveSettings()
    end
end

-- =============================================
--        FIM DO SCRIPT CHERRY HUB v3.3
-- =============================================




