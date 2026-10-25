-- Carrega a biblioteca Fluent
local Fluent = loadstring(game:HttpGet("https://github.com/dawid-scripts/Fluent/releases/latest/download/main.lua"))()

local Window = Fluent:CreateWindow({
    Title = "Savannah Life | Snow Xit",
    SubTitle = "https://discord.gg/snowxit",
    TabWidth = 130,
    Size = UDim2.fromOffset(550, 330),
    Acrylic = false,
    Theme = "Darker"
})

local ReplicatedStorage = game:GetService("ReplicatedStorage")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local StarterGui = game:GetService("StarterGui")
local Lighting = game:GetService("Lighting")
local CollectionService = game:GetService("CollectionService")

local LocalPlayer = Players.LocalPlayer
local ScreenGui = Instance.new("ScreenGui")
local MinimizeButton = Instance.new("ImageButton")
local ButtonCorner = Instance.new("UICorner")

ScreenGui.Name = "MobileMinimize"
ScreenGui.Parent = gethui()
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

MinimizeButton.Parent = ScreenGui
MinimizeButton.BackgroundColor3 = Color3.new(1, 1, 1)
MinimizeButton.BorderColor3 = Color3.new(1, 1, 1)
MinimizeButton.BorderSizePixel = 0
MinimizeButton.Position = UDim2.new(0, 584, 0, 14)

local originalSize = UDim2.new(0, 45, 0, 45)
MinimizeButton.Size = originalSize + UDim2.new(0, originalSize.X.Offset * 0.15, 0, originalSize.Y.Offset * 0.15)
MinimizeButton.Image = "rbxassetid://96505559272717"

ButtonCorner.CornerRadius = UDim.new(0.2, 0)
ButtonCorner.Parent = MinimizeButton

local dragging = false
local dragInput, touchPos, buttonPos

MinimizeButton.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        touchPos = input.Position
        buttonPos = MinimizeButton.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then dragging = false end
        end)
    end
end)

MinimizeButton.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput and dragging then
        local delta = input.Position - touchPos
        MinimizeButton.Position = UDim2.new(
            buttonPos.X.Scale, buttonPos.X.Offset + delta.X,
            buttonPos.Y.Scale, buttonPos.Y.Offset + delta.Y
        )
    end
end)

MinimizeButton.MouseButton1Click:Connect(function() Window:Minimize() end)

-- Tabs
local PlayerTab = Window:AddTab({Title = "Player", Icon = "user"})
local StatusTab = Window:AddTab({Title = "Status", Icon = "heart"})
local AttackTab = Window:AddTab({Title = "Attack", Icon = "crosshair"})
local AutoFarmTab = Window:AddTab({Title = "Auto Farm", Icon = "settings"})
local MiscTab = Window:AddTab({Title = "MISC", Icon = "cog"})
local ServersTab = Window:AddTab({Title = "Servidores", Icon = "server"})
local CreditTab = Window:AddTab({Title = "Créditos", Icon = "users"})

-- ==========================================================
-- HELPER DE SLIDER PRO
-- ==========================================================
local isMobile = UserInputService.TouchEnabled
local debugUIEnabled = false

local function AddSliderPro(tab, id, config)
    local currentValue = config.Default
    local step = config.Step or 1

    local slider = tab:AddSlider(id, {
        Title = config.Title,
        Description = config.Description,
        Default = config.Default,
        Min = config.Min,
        Max = config.Max,
        Rounding = config.Rounding,
        Callback = function(value)
            currentValue = tonumber(value) or value
            if config.Callback then config.Callback(currentValue) end
        end
    })

    if config.Callback and config.Default ~= nil then
        pcall(config.Callback, config.Default)
    end

    if isMobile then
        tab:AddButton({
            Title = "➖ " .. config.Title .. " (-" .. step .. ")",
            Description = "Toque para diminuir",
            Callback = function()
                local novo = currentValue - step
                if novo < config.Min then novo = config.Min end
                currentValue = novo
                if config.Callback then config.Callback(novo) end
                pcall(function() slider:SetValue(novo) end)
            end
        })
        tab:AddButton({
            Title = "➕ " .. config.Title .. " (+" .. step .. ")",
            Description = "Toque para aumentar",
            Callback = function()
                local novo = currentValue + step
                if novo > config.Max then novo = config.Max end
                currentValue = novo
                if config.Callback then config.Callback(novo) end
                pcall(function() slider:SetValue(novo) end)
            end
        })
    end

    return slider
end

-- Player
do
    PlayerTab:AddSection("Anti HIT")

    PlayerTab:AddButton({
        Title = "Anti Dano de Queda",
        Callback = function()
            local ev = ReplicatedStorage:FindFirstChild("PlayerDamageSelfRemoteEvent")
            if ev then ev:Destroy() end
        end
    })

    local cframes = {}
    local desyncEnabled = false

    RunService.Heartbeat:Connect(function()
        if not desyncEnabled then return end
        local character = LocalPlayer.Character
        if character and character:FindFirstChild("HumanoidRootPart") then
            cframes.client = character.HumanoidRootPart.CFrame
            cframes.server = cframes.client
            local desyncCFrame = cframes.client + Vector3.new(0, -30, 0)
            character.HumanoidRootPart.CFrame = desyncCFrame
            cframes.desync = desyncCFrame
        else
            cframes.desync = nil
            cframes.client = nil
            cframes.server = nil
        end
    end)

    RunService:BindToRenderStep("New Frame Start", Enum.RenderPriority.First.Value, function()
        if not desyncEnabled then return end
        if cframes.client and LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            LocalPlayer.Character.HumanoidRootPart.CFrame = cframes.client
        end
    end)

    PlayerTab:AddToggle("AntiHit", {
        Title = "Anti Hit (Invisible)",
        Default = false,
        Callback = function(value)
            desyncEnabled = value
            local character = LocalPlayer.Character
            if not character or not character:FindFirstChild("HumanoidRootPart") then return end
            local hrp = character.HumanoidRootPart
            if not value then
                if cframes.client then hrp.CFrame = cframes.client
                elseif cframes.server then hrp.CFrame = cframes.server end
                cframes.desync = nil
                cframes.client = nil
                cframes.server = nil
            end
        end
    })

    PlayerTab:AddButton({
        Title = "Voltar pro Solo",
        Callback = function()
            local character = LocalPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                local hrp = character.HumanoidRootPart
                hrp.CFrame = hrp.CFrame + Vector3.new(0, 30, 0)
            end
        end
    })

    local carcassEspEnabled = false
    local carcassStorageName = "CarcassesStorageModel"
    local carcassConnections = {}
    local carcassEspCache = {}

    local function aplicarESPCarcaca(carcaca)
        if not carcassEspEnabled or carcaca:FindFirstChild("SnowCarcass_ESP") then return end
        local pRef = carcaca:IsA("BasePart") and carcaca or carcaca:FindFirstChildWhichIsA("BasePart")
        if not pRef then return end
        local holder = Instance.new("Folder")
        holder.Name = "SnowCarcass_ESP"
        holder.Parent = carcaca
        local box = Instance.new("SelectionBox")
        box.Adornee = pRef
        box.Color3 = Color3.fromRGB(0, 255, 127)
        box.LineThickness = 0.05
        box.Transparency = 0.3
        box.SurfaceTransparency = 0.8
        box.Parent = holder
        local tGui = Instance.new("BillboardGui")
        tGui.Adornee = pRef
        tGui.AlwaysOnTop = true
        tGui.Size = UDim2.new(0, 200, 0, 50)
        tGui.StudsOffset = Vector3.new(0, 4, 0)
        tGui.Parent = holder
        local label = Instance.new("TextLabel")
        label.Parent = tGui
        label.BackgroundTransparency = 1
        label.Size = UDim2.new(1, 0, 1, 0)
        label.TextColor3 = Color3.fromRGB(0, 255, 127)
        label.TextSize = 16
        label.Font = Enum.Font.RobotoMono
        label.TextStrokeTransparency = 0
        carcassEspCache[carcaca] = {holder = holder, label = label, part = pRef}
    end

    task.spawn(function()
        while true do
            if carcassEspEnabled then
                local char = LocalPlayer.Character
                if char and char:FindFirstChild("HumanoidRootPart") then
                    for carcaca, data in pairs(carcassEspCache) do
                        if carcaca and carcaca.Parent and data.part and data.part.Parent then
                            local dist = math.floor((char.HumanoidRootPart.Position - data.part.Position).Magnitude)
                            data.label.Text = "🥩 " .. string.upper(carcaca.Name) .. "\n[" .. dist .. "m]"
                        else
                            carcassEspCache[carcaca] = nil
                        end
                    end
                end
            end
            task.wait(1)
        end
    end)

    local function updateCarcassESP()
        for _, conn in pairs(carcassConnections) do conn:Disconnect() end
        carcassConnections = {}
        carcassEspCache = {}
        if not carcassEspEnabled then
            local storage = workspace:FindFirstChild(carcassStorageName)
            if storage then
                for _, item in pairs(storage:GetChildren()) do
                    local h = item:FindFirstChild("SnowCarcass_ESP")
                    if h then h:Destroy() end
                end
            end
            return
        end
        local storage = workspace:FindFirstChild(carcassStorageName)
        if storage then
            for _, item in pairs(storage:GetChildren()) do aplicarESPCarcaca(item) end
            table.insert(carcassConnections, storage.ChildAdded:Connect(aplicarESPCarcaca))
        end
    end

    local showPlayerMarkers = false
    local espConnections = {}

    local function updateESP()
        for _, p in ipairs(Players:GetPlayers()) do
            if p.Character then
                if p.Character:FindFirstChild("SnowESP_Highlight") then p.Character.SnowESP_Highlight:Destroy() end
                local h = p.Character:FindFirstChild("Head")
                if h and h:FindFirstChild("SnowESP_Name") then h.SnowESP_Name:Destroy() end
            end
        end
        if not showPlayerMarkers then
            for _, conn in pairs(espConnections) do conn:Disconnect() end
            espConnections = {}
            return
        end
        local function applyToPlayer(plr)
            if plr == LocalPlayer then return end
            local function setup(character)
                if not showPlayerMarkers then return end
                local highlight = Instance.new("Highlight")
                highlight.Name = "SnowESP_Highlight"
                highlight.FillTransparency = 1
                highlight.OutlineColor = Color3.fromRGB(255, 0, 0)
                highlight.Parent = character
                local head = character:WaitForChild("Head", 5)
                if head then
                    if head:FindFirstChild("SnowESP_Name") then head.SnowESP_Name:Destroy() end
                    local billboard = Instance.new("BillboardGui", head)
                    billboard.Name = "SnowESP_Name"
                    billboard.Size = UDim2.new(0, 150, 0, 30)
                    billboard.StudsOffset = Vector3.new(0, 2.5, 0)
                    billboard.AlwaysOnTop = true
                    local nameLabel = Instance.new("TextLabel", billboard)
                    nameLabel.BackgroundTransparency = 1
                    nameLabel.Size = UDim2.new(1, 0, 1, 0)
                    nameLabel.Text = plr.DisplayName or plr.Name
                    nameLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
                    nameLabel.TextSize = 11
                    nameLabel.Font = Enum.Font.SourceSans
                    nameLabel.TextStrokeTransparency = 0.8
                end
            end
            if plr.Character then task.spawn(setup, plr.Character) end
            table.insert(espConnections, plr.CharacterAdded:Connect(setup))
        end
        for _, plr in ipairs(Players:GetPlayers()) do applyToPlayer(plr) end
        table.insert(espConnections, Players.PlayerAdded:Connect(applyToPlayer))
    end

    local enabledFunc = false
    Lighting.ClockTime = 10
    Lighting:GetPropertyChangedSignal("ClockTime"):Connect(function()
        if enabledFunc and Lighting.ClockTime ~= 10 then Lighting.ClockTime = 10 end
    end)

    PlayerTab:AddSection("Visuais")

    PlayerTab:AddToggle("CarcassESP", {
        Title = "ESP de Carcaças",
        Default = false,
        Callback = function(value) carcassEspEnabled = value; updateCarcassESP() end
    })

    PlayerTab:AddToggle("ShowPlayerMarkers", {
        Title = "Esp jogadores",
        Default = false,
        Callback = function(value) showPlayerMarkers = value; updateESP() end
    })

    PlayerTab:AddToggle("AlwaysDay", {
        Title = "Sempre de Dia",
        Default = false,
        Callback = function(value)
            enabledFunc = value
            if value then Lighting.ClockTime = 10 end
        end
    })

    PlayerTab:AddSection("Speed Hack")

    local speedEnabled = false
    local targetSpeed = 30
    local originalSpeed = 16
    local bypassActive = false

    local function modifyTargetSpeed()
        local character = LocalPlayer.Character
        if character then
            if character:GetAttribute("TargetSpeed") then
                character:SetAttribute("TargetSpeed", targetSpeed)
                return true
            end
            for _, child in ipairs(character:GetDescendants()) do
                if child:GetAttribute("TargetSpeed") then
                    child:SetAttribute("TargetSpeed", targetSpeed)
                    return true
                end
            end
        end
        return false
    end

    local function speedBypass()
        while speedEnabled and bypassActive do
            local targetModified = modifyTargetSpeed()
            if not targetModified then
                local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then humanoid.WalkSpeed = targetSpeed end
            end
            task.wait(0.1)
        end
    end

    PlayerTab:AddToggle("SpeedHack", {
        Title = "Speed Hack",
        Default = false,
        Callback = function(value)
            speedEnabled = value
            bypassActive = value
            if value then
                local humanoid = LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then originalSpeed = humanoid.WalkSpeed end
                local character = LocalPlayer.Character
                if character and character:GetAttribute("TargetSpeed") then
                    originalSpeed = character:GetAttribute("TargetSpeed")
                end
                task.spawn(speedBypass)
            else
                local character = LocalPlayer.Character
                if character then
                    if character:GetAttribute("TargetSpeed") then
                        character:SetAttribute("TargetSpeed", originalSpeed)
                    end
                    local humanoid = character:FindFirstChildOfClass("Humanoid")
                    if humanoid then humanoid.WalkSpeed = originalSpeed end
                end
            end
        end
    })

    AddSliderPro(PlayerTab, "SpeedValue", {
        Title = "Speed Value",
        Description = "Ajuste a velocidade",
        Default = 30, Min = 16, Max = 85, Rounding = 1,
        Callback = function(value)
            targetSpeed = value
            if speedEnabled then
                modifyTargetSpeed()
                local humanoid = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
                if humanoid then humanoid.WalkSpeed = value end
            end
        end
    })

    local antiGrabConfig = {delay = 0.1, enabled = false, active = false}

    local function AutoUnlatchLoop()
        while antiGrabConfig.active do
            if antiGrabConfig.enabled then
                local gui = LocalPlayer.PlayerGui:FindFirstChild("ExitLatchScreenGui")
                if gui and gui.Enabled then
                    pcall(function()
                        ReplicatedStorage.LatchedPlayersNotifyRemoteEvent:FireServer()
                    end)
                end
            end
            task.wait(antiGrabConfig.delay)
        end
    end

    PlayerTab:AddSection("Anti Grab")

    PlayerTab:AddToggle("Toggle_AntiGrab_New", {
        Title = "Anti Grab",
        Default = false,
        Callback = function(state)
            antiGrabConfig.enabled = state
            if state and not antiGrabConfig.active then
                antiGrabConfig.active = true
                task.spawn(AutoUnlatchLoop)
            end
        end
    })

    AddSliderPro(PlayerTab, "Slider_AntiGrab_Delay", {
        Title = "Velocidade do Anti Grab",
        Description = "1 a 10 (10 = Mais rápido)",
        Default = 10, Min = 1, Max = 10, Rounding = 0, Step = 1,
        Callback = function(val) 
            antiGrabConfig.delay = 1 / val 
        end
    })
end

-- Status
do
    StatusTab:AddSection("Infinite Stats")

    local StatsEnabled = {Oxygen = false, Stamina = false, Water = false, Food = false}
    local protectionConnections = {}
    local lastUpdate = 0
    local UPDATE_INTERVAL = 0.5

    local function UpdateCharacterStats(char)
        if not char then return end
        for stat, enabled in pairs(StatsEnabled) do
            if enabled then
                pcall(function()
                    char:SetAttribute(stat, 100)
                    local humanoid = char:FindFirstChildOfClass("Humanoid")
                    if humanoid and humanoid:GetAttribute(stat) then
                        humanoid:SetAttribute(stat, 100)
                    end
                end)
            end
        end
    end

    local function ToggleInfiniteStats(stat, value)
        StatsEnabled[stat] = value
        for _, conn in ipairs(protectionConnections) do conn:Disconnect() end
        protectionConnections = {}
        local anyEnabled = false
        for _, enabled in pairs(StatsEnabled) do
            if enabled then anyEnabled = true; break end
        end
        if anyEnabled then
            UpdateCharacterStats(LocalPlayer.Character)
            table.insert(protectionConnections, LocalPlayer.CharacterAdded:Connect(function(newChar)
                UpdateCharacterStats(newChar)
            end))
            table.insert(protectionConnections, RunService.Stepped:Connect(function()
                local now = tick()
                if now - lastUpdate >= UPDATE_INTERVAL then
                    lastUpdate = now
                    UpdateCharacterStats(LocalPlayer.Character)
                end
            end))
        end
    end

    StatusTab:AddToggle("InfiniteOxygen", {
        Title = "Infinite Oxygen", Default = false,
        Callback = function(value) ToggleInfiniteStats("Oxygen", value) end
    })

    StatusTab:AddToggle("InfiniteStamina", {
        Title = "Infinite Stamina", Default = false,
        Callback = function(value) ToggleInfiniteStats("Stamina", value) end
    })
end

-- Ataque
do
    AttackTab:AddSection("Kill Aura")

    local killAuraEnabled = false
    local attackRange = 20
    local killAuraConnection = nil
    local lastGlobalAttack = 0
    local baseCooldown = 0.07
    local maxAttacksPerSecond = 18
    local attackHistory = {}
    local maxHistorySize = 30
    local recentHits = {}
    local hitDebounce = 0.25
    local burstMode = false
    local burstStartTime = 0
    local burstDuration = 1.0
    local burstCooldown = 1.0
    local mainLoopInterval = 0.08
    local lastMainLoop = 0
    local crowdHistory = {}
    local maxCrowdHistory = 10

    local whitelistedUserIds = {}

    local function isModelWhitelisted(model)
        if not model then return false end
        local plr = Players:GetPlayerFromCharacter(model)
        if not plr then return false end
        for _, uid in ipairs(whitelistedUserIds) do
            if uid == plr.UserId then return true end
        end
        return false
    end

    local function getRandomDelay(base, variation)
        return base + (math.random() - 0.5) * variation
    end

    local function cleanAttackHistory()
        local now = tick()
        local newHistory = {}
        for _, timestamp in ipairs(attackHistory) do
            if now - timestamp < 1 then table.insert(newHistory, timestamp) end
        end
        attackHistory = newHistory
    end

    local function canAttack()
        local now = tick()
        local recentAttacks = 0
        for _, timestamp in ipairs(attackHistory) do
            if now - timestamp < 0.5 then recentAttacks = recentAttacks + 1 end
        end
        local crowdFactor = math.min(recentAttacks / 8, 1)
        local adaptiveCooldown = baseCooldown + (crowdFactor * 0.03)
        local actualCooldown = getRandomDelay(adaptiveCooldown, 0.015)
        if now - lastGlobalAttack < actualCooldown then return false end
        cleanAttackHistory()
        local currentLimit = maxAttacksPerSecond - math.floor(crowdFactor * 4)
        if burstMode then currentLimit = currentLimit + 4 end
        if #attackHistory >= currentLimit then return false end
        return true
    end

    local function executeAttack(targetHumanoid, isBurst)
        if not canAttack() then return false end
        local now = tick()
        local actualDebounce = isBurst and (hitDebounce * 0.5) or hitDebounce
        if recentHits[targetHumanoid] and now - recentHits[targetHumanoid] < actualDebounce then
            return false
        end
        task.spawn(function()
            task.wait(getRandomDelay(0, 0.012))
            pcall(function()
                local Event = ReplicatedStorage:FindFirstChild("AttackHandlerRemoteEvent")
                if Event then Event:FireServer(targetHumanoid) end
            end)
        end)
        lastGlobalAttack = now
        recentHits[targetHumanoid] = now
        table.insert(attackHistory, now)
        if #attackHistory > maxHistorySize then table.remove(attackHistory, 1) end
        return true
    end

    local function shouldStartBurst(targetCount)
        local now = tick()
        local burstThreshold = targetCount >= 5 and 3 or 2
        if targetCount >= burstThreshold and not burstMode and now - burstStartTime > burstCooldown then
            local dynamicCooldown = burstCooldown + (targetCount > 4 and 0.3 or 0)
            if now - burstStartTime > dynamicCooldown then
                burstMode = true; burstStartTime = now; return true
            end
        end
        local dynamicDuration = targetCount > 4 and (burstDuration * 0.7) or burstDuration
        if burstMode and now - burstStartTime > dynamicDuration then
            burstMode = false; burstStartTime = now
        end
        return false
    end

    local function findNearbyHumanoids()
        local character = LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return {} end
        local targets = {}
        local characterPos = character.HumanoidRootPart.Position
        for _, model in ipairs(workspace:GetChildren()) do
            if model ~= character then
                local hum = model:FindFirstChildOfClass("Humanoid")
                local hrp = model:FindFirstChild("HumanoidRootPart")
                if hum and hrp and hum.Health > 0 then
                    if not isModelWhitelisted(model) then
                        local distance = (hrp.Position - characterPos).Magnitude
                        if distance <= attackRange then
                            table.insert(targets, {humanoid = hum, distance = distance, priority = math.random()})
                        end
                    end
                end
            end
        end
        if #targets > 1 then
            table.sort(targets, function(a, b)
                local scoreA = (1 / a.distance) + (a.priority * 0.1)
                local scoreB = (1 / b.distance) + (b.priority * 0.1)
                return scoreA > scoreB
            end)
        end
        local result = {}
        for _, target in ipairs(targets) do table.insert(result, target.humanoid) end
        return result
    end

    local function updateCrowdHistory(targetCount)
        table.insert(crowdHistory, targetCount)
        if #crowdHistory > maxCrowdHistory then table.remove(crowdHistory, 1) end
    end

    local function getAverageCrowd()
        if #crowdHistory == 0 then return 0 end
        local sum = 0
        for _, count in ipairs(crowdHistory) do sum = sum + count end
        return sum / #crowdHistory
    end

    local function killAuraLoop()
        local now = tick()
        if now - lastMainLoop < mainLoopInterval then return end
        lastMainLoop = now
        local targets = findNearbyHumanoids()
        local targetCount = #targets
        if targetCount == 0 then return end
        updateCrowdHistory(targetCount)
        local avgCrowd = getAverageCrowd()
        local isInCrowd = avgCrowd > 3
        shouldStartBurst(targetCount)
        local maxTargetsThisCycle
        if isInCrowd then
            maxTargetsThisCycle = 1
        else
            if burstMode then maxTargetsThisCycle = math.min(targetCount, 3)
            elseif targetCount == 1 then maxTargetsThisCycle = 1
            else maxTargetsThisCycle = math.min(targetCount, 2) end
        end
        local attacksExecuted = 0
        for i = 1, maxTargetsThisCycle do
            if attacksExecuted >= maxTargetsThisCycle then break end
            local target = targets[i]
            if target and target.Parent then
                if executeAttack(target, burstMode) then
                    attacksExecuted = attacksExecuted + 1
                    if attacksExecuted < maxTargetsThisCycle then
                        local crowdDelay = isInCrowd and 0.015 or 0.008
                        task.wait(getRandomDelay(crowdDelay, 0.004))
                    end
                end
            end
        end
    end

    -- UI Whitelist
    AttackTab:AddSection("Whitelist")

    local selectedPlayerForWhitelist = nil

    local function getPlayerList()
        local list = {}
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer then table.insert(list, player.Name) end
        end
        return list
    end

    local whitelistDropdown = AttackTab:AddDropdown("WhitelistPlayerSelect", {
        Title = "Selecionar Jogador",
        Description = "Escolha um jogador para adicionar/remover da whitelist",
        Values = getPlayerList(),
        Multi = false,
        Default = nil,
        Callback = function(value) selectedPlayerForWhitelist = value end
    })

    Players.PlayerAdded:Connect(function()
        task.wait(0.3)
        pcall(function() whitelistDropdown:Refresh(getPlayerList()) end)
    end)
    Players.PlayerRemoving:Connect(function()
        task.wait(0.3)
        pcall(function() whitelistDropdown:Refresh(getPlayerList()) end)
    end)

    AttackTab:AddButton({
        Title = "Adicionar à Whitelist",
        Description = "Adiciona o jogador selecionado à whitelist",
        Callback = function()
            if not selectedPlayerForWhitelist then
                Fluent:Notify({Title="Whitelist", Content="Selecione um jogador primeiro!", Duration=3}); return
            end
            local targetPlayer = Players:FindFirstChild(selectedPlayerForWhitelist)
            if not targetPlayer then
                Fluent:Notify({Title="Whitelist", Content="Jogador não encontrado!", Duration=3}); return
            end
            for _, uid in ipairs(whitelistedUserIds) do
                if uid == targetPlayer.UserId then
                    Fluent:Notify({Title="Whitelist", Content=selectedPlayerForWhitelist.." já está na whitelist!", Duration=3}); return
                end
            end
            table.insert(whitelistedUserIds, targetPlayer.UserId)
            Fluent:Notify({Title="Whitelist", Content=selectedPlayerForWhitelist.." adicionado à whitelist!", Duration=3})
        end
    })

    AttackTab:AddButton({
        Title = "Remover da Whitelist",
        Description = "Remove o jogador selecionado da whitelist",
        Callback = function()
            if not selectedPlayerForWhitelist then
                Fluent:Notify({Title="Whitelist", Content="Selecione um jogador primeiro!", Duration=3}); return
            end
            local targetPlayer = Players:FindFirstChild(selectedPlayerForWhitelist)
            if not targetPlayer then
                Fluent:Notify({Title="Whitelist", Content="Jogador não encontrado!", Duration=3}); return
            end
            for i, uid in ipairs(whitelistedUserIds) do
                if uid == targetPlayer.UserId then
                    table.remove(whitelistedUserIds, i)
                    Fluent:Notify({Title="Whitelist", Content=selectedPlayerForWhitelist.." removido da whitelist!", Duration=3}); return
                end
            end
            Fluent:Notify({Title="Whitelist", Content="Jogador não está na whitelist!", Duration=3})
        end
    })

    AttackTab:AddButton({
        Title = "Limpar Whitelist",
        Description = "Remove todos os jogadores da whitelist",
        Callback = function()
            whitelistedUserIds = {}
            Fluent:Notify({Title="Whitelist", Content="Whitelist limpa!", Duration=3})
        end
    })

    AttackTab:AddToggle("KillAura", {
        Title = "Kill Aura",
        Default = false,
        Callback = function(value)
            killAuraEnabled = value
            if value then
                killAuraConnection = RunService.Heartbeat:Connect(killAuraLoop)
                task.spawn(function()
                    while killAuraEnabled do
                        task.wait(3)
                        local newRecentHits = {}
                        for hum, time in pairs(recentHits) do
                            if hum.Parent and hum.Health > 0 then newRecentHits[hum] = time end
                        end
                        recentHits = newRecentHits
                    end
                end)
            elseif killAuraConnection then
                killAuraConnection:Disconnect()
                killAuraConnection = nil
                lastGlobalAttack = 0
                attackHistory = {}
                recentHits = {}
                burstMode = false
                burstStartTime = 0
                crowdHistory = {}
            end
        end
    })

    AddSliderPro(AttackTab, "AttackRange", {
        Title = "Attack Range",
        Description = "Ajuste o alcance",
        Default = 20, Min = 5, Max = 50, Rounding = 1,
        Callback = function(value) attackRange = tonumber(value) end
    })

    -- Auto Grab
    AttackTab:AddSection("Auto Grab")
    local autoGrabEnabled = false
    local HEIGHT_OFFSET = 8
    local MAX_SPEED = 67
    local MIN_DISTANCE = 4
    local MAX_DISTANCE = 100
    local autoGrabConnection = nil

    local function findClosestPlayer()
        local closestPlayer = nil
        local closestDistance = MAX_DISTANCE
        local character = LocalPlayer.Character
        if not character or not character:FindFirstChild("HumanoidRootPart") then return nil end
        local characterPos = character.HumanoidRootPart.Position
        for _, player in pairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character
               and player.Character:FindFirstChild("HumanoidRootPart")
               and player.Character:FindFirstChild("Humanoid")
               and player.Character.Humanoid.Health > 0
               and not isModelWhitelisted(player.Character) then
                local targetRoot = player.Character.HumanoidRootPart
                local distance = (characterPos - targetRoot.Position).Magnitude
                if distance < closestDistance then
                    closestDistance = distance
                    closestPlayer = player
                end
            end
        end
        return closestPlayer
    end

    local function executeAutoGrab()
        if not autoGrabEnabled then return end
        local closestPlayer = findClosestPlayer()
        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local character = LocalPlayer.Character
            if not character or not character:FindFirstChild("HumanoidRootPart") then return end
            local humanoidRoot = character.HumanoidRootPart
            local targetRoot = closestPlayer.Character.HumanoidRootPart
            local targetPos = targetRoot.Position + Vector3.new(0, HEIGHT_OFFSET, 0)
            local distance = (targetPos - humanoidRoot.Position).Magnitude
            if distance < MIN_DISTANCE then
                humanoidRoot.Velocity = Vector3.new(0, 0, 0)
                return
            end
            local moveDir = (targetPos - humanoidRoot.Position).Unit
            humanoidRoot.Velocity = moveDir * MAX_SPEED
            humanoidRoot.CFrame = CFrame.lookAt(humanoidRoot.Position, humanoidRoot.Position + moveDir)
        else
            local character = LocalPlayer.Character
            if character and character:FindFirstChild("HumanoidRootPart") then
                character.HumanoidRootPart.Velocity = Vector3.new(0, 0, 0)
            end
        end
    end

    AttackTab:AddToggle("AutoGrab", {
        Title = "Auto Grab",
        Default = false,
        Callback = function(value)
            autoGrabEnabled = value
            if value then
                autoGrabConnection = RunService.Heartbeat:Connect(executeAutoGrab)
            else
                if autoGrabConnection then autoGrabConnection:Disconnect(); autoGrabConnection = nil end
                local character = LocalPlayer.Character
                if character and character:FindFirstChild("HumanoidRootPart") then
                    character.HumanoidRootPart.Velocity = Vector3.new(0, 0, 0)
                end
            end
        end
    })

    AddSliderPro(AttackTab, "GrabRange", {
        Title = "Distancia do Auto Grab",
        Description = "Ajuste a distância máxima",
        Default = 100, Min = 50, Max = 200, Rounding = 1,
        Callback = function(value) MAX_DISTANCE = value end
    })

    -- ==========================================
    -- HITBOX NORMAL DE ATAQUE (M1 / CLIQUE)
    -- ==========================================
    AttackTab:AddSection("Hitbox Normal")

    local hitboxEnabled = false
    local hitboxRange = 25
    local hitboxLast = 0
    local AttackRemote = ReplicatedStorage:WaitForChild("AttackHandlerRemoteEvent", 10)

    local function fireHitboxAttack()
        if not hitboxEnabled or os.clock() - hitboxLast < 0.20 then return end
        hitboxLast = os.clock()

        local char = LocalPlayer.Character
        local myRoot = char and char:FindFirstChild("HumanoidRootPart")
        if not myRoot then return end

        local animals = CollectionService:GetTagged("Animals")

        if #animals == 0 then
            animals = {}
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") then
                    table.insert(animals, obj)
                end
            end
        end

        for _, model in ipairs(animals) do
            if model ~= char and model:IsA("Model") and not isModelWhitelisted(model) then
                local hum = model:FindFirstChildOfClass("Humanoid")
                local root = model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart

                if hum and hum.Health > 0 and root then
                    if (root.Position - myRoot.Position).Magnitude <= hitboxRange then
                        pcall(function()
                            if AttackRemote then AttackRemote:FireServer(hum) end
                        end)
                        break
                    end
                end
            end
        end
    end

    AttackTab:AddToggle("HitboxToggle", {
        Title = "Ativar Hitbox",
        Description = "Dá dano no inimigo mais próximo (PC: Clique / Mobile: Botão Nativo de Ataque)",
        Default = false,
        Callback = function(val) hitboxEnabled = val end
    })

    AddSliderPro(AttackTab, "HitboxRangeSlider", {
        Title = "Alcance do Hitbox",
        Description = "Distância máxima para dar o ataque",
        Default = 25, Min = 10, Max = 50, Rounding = 1,
        Callback = function(val) hitboxRange = val end
    })

    UserInputService.InputBegan:Connect(function(input, processed)
        if not processed and input.UserInputType == Enum.UserInputType.MouseButton1 then
            fireHitboxAttack()
        end
    end)

    -- ==========================================
    -- HITBOX GRAB / POUNCE (TECLA F)
    -- ==========================================
    AttackTab:AddSection("Hitbox Grab / Pounce")

    local pounceEnabled = false
    local pounceRange = 50
    local pounceRemote = ReplicatedStorage:WaitForChild("SpecialAttackRemoteEvent_PounceAttack", 10)
    local lastPounceTime = 0

    local function firePounce()
        if not pounceEnabled or os.clock() - lastPounceTime < 0.35 then return end
        lastPounceTime = os.clock()

        local char = LocalPlayer.Character
        local myRoot = char and char:FindFirstChild("HumanoidRootPart")
        if not myRoot then return end

        local animals = CollectionService:GetTagged("Animals")
        if #animals == 0 then
            animals = {}
            for _, obj in ipairs(workspace:GetChildren()) do
                if obj:IsA("Model") and obj:FindFirstChildOfClass("Humanoid") then
                    table.insert(animals, obj)
                end
            end
        end

        for _, model in ipairs(animals) do
            if model ~= char and model:IsA("Model") and not isModelWhitelisted(model) then
                local hum = model:FindFirstChildOfClass("Humanoid")
                local root = model:FindFirstChild("HumanoidRootPart") or model.PrimaryPart
                local folder = model:FindFirstChild("RagdollColliderParts")
                local rag = (folder and folder:FindFirstChild("RagdollColliderRootPart")) or root

                if hum and hum.Health > 0 and root and rag then
                    if (root.Position - myRoot.Position).Magnitude <= pounceRange then
                        pcall(function()
                            if pounceRemote then pounceRemote:FireServer(hum, rag) end
                        end)
                        break
                    end
                end
            end
        end
    end

    AttackTab:AddToggle("PounceToggle", {
        Title = "Ativar Grab",
        Description = "Puxa e gruda no inimigo mais próximo (PC: F / Mobile: Botão Especial)",
        Default = false,
        Callback = function(val) pounceEnabled = val end
    })

    AddSliderPro(AttackTab, "PounceRange", {
        Title = "Distância do Grab",
        Description = "Alcance do puxão",
        Default = 50, Min = 10, Max = 150, Rounding = 1,
        Callback = function(val) pounceRange = val end
    })

    UserInputService.InputBegan:Connect(function(input, processed)
        if not processed and input.KeyCode == Enum.KeyCode.F then
            firePounce()
        end
    end)

    -- ==========================================
    -- HOOK NOS BOTÕES NATIVOS DO JOGO (MOBILE)
    -- ==========================================
    if isMobile then
        local function getButtonFromObject(obj)
            local btn = obj
            while btn and not (btn:IsA("ImageButton") or btn:IsA("TextButton")) do
                btn = btn.Parent
            end
            return btn
        end

        UserInputService.InputBegan:Connect(function(input, gameProcessed)
            if input.UserInputType ~= Enum.UserInputType.Touch then return end
            
            if gameProcessed then
                local pos = input.Position
                local guiObjects = LocalPlayer.PlayerGui:GetGuiObjectsAtPosition(pos.X, pos.Y)
                
                for _, gui in ipairs(guiObjects) do
                    local btn = getButtonFromObject(gui)
                    if btn and btn.Visible then
                        local btnName = string.lower(btn.Name)
                        local parentName = btn.Parent and string.lower(btn.Parent.Name) or ""
                        local fullName = btnName .. " " .. parentName

                        -- DEBUG DE BOTÕES
                        if debugUIEnabled then
                            print("[SnowXit Debug] Botão tocado:", fullName, "| Tamanho:", btn.AbsoluteSize.X, "x", btn.AbsoluteSize.Y)
                        end
                        
                        -- 1. Tenta identificar pelos nomes mais comuns
                        local isAttack = string.find(fullName, "attack") or string.find(fullName, "hit") or string.find(fullName, "m1") or string.find(fullName, "punch") or string.find(fullName, "bite") or string.find(fullName, "paw")
                        local isSpecial = string.find(fullName, "special") or string.find(fullName, "grab") or string.find(fullName, "pounce") or string.find(fullName, "claw") or string.find(fullName, "ability") or string.find(fullName, "skill")
                        
                        -- 2. Se não achar pelo nome, usa posição e tamanho exatos
                        if not isAttack and not isSpecial then
                            local cam = workspace.CurrentCamera
                            if cam then
                                local viewport = cam.ViewportSize
                                local btnPos = btn.AbsolutePosition
                                local btnSize = btn.AbsoluteSize
                                local btnCenterX = btnPos.X + btnSize.X / 2
                                local btnCenterY = btnPos.Y + btnSize.Y / 2
                                
                                -- Verifica se está no canto inferior direito
                                if btnCenterX > viewport.X * 0.7 and btnCenterY > viewport.Y * 0.6 then
                                    -- Botão de Ataque Normal (Pata) é o maior de todos
                                    if btnSize.X > 100 and btnSize.Y > 100 then
                                        isAttack = true
                                    -- Botão Especial (Garra) é médio-grande e fica acima da pata
                                    elseif btnSize.X > 65 and btnSize.X <= 100 and btnSize.Y > 65 and btnSize.Y <= 100 then
                                        -- Filtro extra para não pegar Pulo/Corrida (Y muito alto)
                                        if btnCenterY > viewport.Y * 0.65 and btnCenterY < viewport.Y * 0.85 then
                                            isSpecial = true
                                        end
                                    end
                                end
                            end
                        end
                        
                        if hitboxEnabled and isAttack then
                            fireHitboxAttack()
                            return
                        end
                        
                        if pounceEnabled and isSpecial then
                            firePounce()
                            return
                        end
                    end
                end
            end
        end)
    end

    -- ESP jogadores
    local espEnabled = false
    local espObjects = {}
    local espUpdateConnection
    local playerAddedConnection
    local playerRemovingConnection
    local lastEspUpdate = 0
    local ESP_UPDATE_INTERVAL = 0.5

    local function createEsp(player)
        if player == LocalPlayer then return end
        if espObjects[player] then espObjects[player]:Destroy(); espObjects[player] = nil end
        if not player.Character then player.CharacterAdded:Wait() end
        local character = player.Character
        if not character:FindFirstChild("HumanoidRootPart") then character:WaitForChild("HumanoidRootPart") end
        local hrp = character.HumanoidRootPart
        local espHolder = Instance.new("Folder")
        espHolder.Name = player.Name .. "_ESP"
        espHolder.Parent = game:GetService("CoreGui")
        local billboard = Instance.new("BillboardGui")
        billboard.Name = "Info"
        billboard.Adornee = hrp
        billboard.Size = UDim2.new(0, 200, 0, 50)
        billboard.StudsOffset = Vector3.new(0, 4, 0)
        billboard.AlwaysOnTop = true
        billboard.MaxDistance = math.huge
        billboard.ExtentsOffset = Vector3.new(0, 0, 0)
        billboard.Parent = espHolder
        local textLabel = Instance.new("TextLabel")
        textLabel.Size = UDim2.new(1, 0, 1, 0)
        textLabel.BackgroundTransparency = 1
        textLabel.TextColor3 = Color3.new(1, 1, 1)
        textLabel.TextStrokeTransparency = 0
        textLabel.TextStrokeColor3 = Color3.new(0, 0, 0)
        textLabel.Text = "Loading..."
        textLabel.Font = Enum.Font.SourceSansBold
        textLabel.TextSize = 14
        textLabel.Parent = billboard
        espObjects[player] = espHolder
        if LocalPlayer.Character and LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then
            local humanoid = character:FindFirstChildOfClass("Humanoid")
            textLabel.Text = string.format("%s\nDist: %.1f\nHP: %d",
                player.Name,
                (hrp.Position - LocalPlayer.Character.HumanoidRootPart.Position).Magnitude,
                humanoid and humanoid.Health or 0)
        end
        player.CharacterAdded:Connect(function(newChar)
            if espEnabled then task.wait(0.5); createEsp(player) end
        end)
    end

    local function updateEsp()
        if not LocalPlayer.Character or not LocalPlayer.Character:FindFirstChild("HumanoidRootPart") then return end
        local localHrp = LocalPlayer.Character.HumanoidRootPart
        for player, espHolder in pairs(espObjects) do
            if player and player.Character and player.Character:FindFirstChild("HumanoidRootPart") then
                local hrp = player.Character.HumanoidRootPart
                local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                if espHolder and espHolder:FindFirstChild("Info") then
                    local distance = (hrp.Position - localHrp.Position).Magnitude
                    espHolder.Info.TextLabel.Text = string.format("%s\nDist: %.1f\nHP: %d",
                        player.Name, distance, humanoid and humanoid.Health or 0)
                    local textSize = math.clamp(14 - (distance / 100), 8, 14)
                    espHolder.Info.TextLabel.TextSize = textSize
                end
            end
        end
    end

    local function toggleEsp(value)
        espEnabled = value
        if value then
            for _, espHolder in pairs(espObjects) do if espHolder then espHolder:Destroy() end end
            espObjects = {}
            for _, player in ipairs(Players:GetPlayers()) do task.spawn(createEsp, player) end
            playerAddedConnection = Players.PlayerAdded:Connect(function(player) task.spawn(createEsp, player) end)
            playerRemovingConnection = Players.PlayerRemoving:Connect(function(player)
                if espObjects[player] then espObjects[player]:Destroy(); espObjects[player] = nil end
            end)
            espUpdateConnection = RunService.Stepped:Connect(function()
                local now = tick()
                if now - lastEspUpdate >= ESP_UPDATE_INTERVAL then
                    lastEspUpdate = now
                    updateEsp()
                end
            end)
        else
            for _, espHolder in pairs(espObjects) do if espHolder then espHolder:Destroy() end end
            espObjects = {}
            if espUpdateConnection then espUpdateConnection:Disconnect(); espUpdateConnection = nil end
            if playerAddedConnection then playerAddedConnection:Disconnect(); playerAddedConnection = nil end
            if playerRemovingConnection then playerRemovingConnection:Disconnect(); playerRemovingConnection = nil end
        end
    end

    UserInputService.WindowFocusReleased:Connect(function()
        if espEnabled then toggleEsp(false) end
    end)

    LocalPlayer.CharacterAdded:Connect(function()
        if killAuraEnabled and killAuraConnection then
            killAuraConnection:Disconnect()
            killAuraConnection = RunService.Heartbeat:Connect(killAuraLoop)
        end
        if espEnabled then
            toggleEsp(false)
            task.wait(0.5)
            toggleEsp(true)
        end
    end)
end

-- Auto Farm
do
    AutoFarmTab:AddSection("Auto Quest")

    local autoQuestEnabled = false
    local runningThread = nil

    local function startAutoQuest()
        local character = LocalPlayer.Character
        if not character then return end
        local rootPart = character:FindFirstChild("HumanoidRootPart")
        if not rootPart then return end
        local velocidade = 30
        local forcaGiro = 5
        while autoQuestEnabled do
            rootPart.AssemblyLinearVelocity = rootPart.CFrame.LookVector * velocidade
            rootPart.CFrame = rootPart.CFrame * CFrame.Angles(0, math.rad(forcaGiro), 0)
            RunService.Heartbeat:Wait()
        end
        rootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
    end

    AutoFarmTab:AddToggle("AutoQuest", {
        Title = "Iniciar Auto Farm (Savannah)",
        Default = false,
        Callback = function(value)
            autoQuestEnabled = value
            if value then
                if not runningThread then runningThread = task.spawn(startAutoQuest) end
            elseif runningThread then
                task.cancel(runningThread)
                runningThread = nil
                local character = LocalPlayer.Character
                if character and character:FindFirstChild("HumanoidRootPart") then
                    character.HumanoidRootPart.Anchored = false
                    character.HumanoidRootPart.AssemblyLinearVelocity = Vector3.new(0, 0, 0)
                end
            end
        end
    })

    local antiAfkEnabled = false
    local afkStartTime = 0
    local afkGui = nil
    local afkThread = nil
    local lastInputTime = os.time()
    local lastAfkAction = 0
    local AFK_ACTION_INTERVAL = 5

    local function CreateAfkGui()
        if afkGui then afkGui:Destroy() end
        afkGui = Instance.new("ScreenGui")
        afkGui.Name = "DeltaAntiAFK"
        afkGui.ResetOnSpawn = false
        afkGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        afkGui.Parent = game:GetService("CoreGui")

        local frame = Instance.new("Frame")
        frame.Name = "MainFrame"
        frame.Size = UDim2.new(0, 260, 0, 70)
        frame.Position = UDim2.new(1, -280, 1, -90)
        frame.BackgroundColor3 = Color3.fromRGB(25, 25, 35)
        frame.BackgroundTransparency = 0.3
        frame.BorderSizePixel = 0
        frame.Parent = afkGui

        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 8)
        corner.Parent = frame

        local stroke = Instance.new("UIStroke")
        stroke.Color = Color3.fromRGB(100, 180, 255)
        stroke.Thickness = 1.5
        stroke.Transparency = 0.3
        stroke.Parent = frame

        local icon = Instance.new("ImageLabel")
        icon.Size = UDim2.new(0, 30, 0, 30)
        icon.Position = UDim2.new(0, 15, 0.5, -15)
        icon.Image = "rbxassetid://3926305904"
        icon.ImageRectOffset = Vector2.new(164, 404)
        icon.ImageRectSize = Vector2.new(36, 36)
        icon.BackgroundTransparency = 1
        icon.Parent = frame

        local timeLabel = Instance.new("TextLabel")
        timeLabel.Name = "TimeLabel"
        timeLabel.Size = UDim2.new(0, 180, 0, 30)
        timeLabel.Position = UDim2.new(0, 60, 0.3, 0)
        timeLabel.BackgroundTransparency = 1
        timeLabel.Font = Enum.Font.GothamMedium
        timeLabel.Text = "00:00"
        timeLabel.TextSize = 24
        timeLabel.TextColor3 = Color3.fromRGB(220, 240, 255)
        timeLabel.TextXAlignment = Enum.TextXAlignment.Left
        timeLabel.Parent = frame

        local statusLabel = Instance.new("TextLabel")
        statusLabel.Size = UDim2.new(0, 180, 0, 20)
        statusLabel.Position = UDim2.new(0, 60, 0.7, 0)
        statusLabel.BackgroundTransparency = 1
        statusLabel.Font = Enum.Font.Gotham
        statusLabel.Text = "AFK Ativo"
        statusLabel.TextSize = 14
        statusLabel.TextColor3 = Color3.fromRGB(150, 255, 150)
        statusLabel.TextXAlignment = Enum.TextXAlignment.Left
        statusLabel.Parent = frame

        frame.Position = UDim2.new(1, 300, 1, -90)
        game:GetService("TweenService"):Create(frame, TweenInfo.new(0.4, Enum.EasingStyle.Quint), {
            Position = UDim2.new(1, -280, 1, -90)
        }):Play()
    end

    local function UpdateAfkTime()
        while antiAfkEnabled and afkGui do
            local afkSeconds = os.time() - afkStartTime
            local minutes = math.floor(afkSeconds / 60)
            local seconds = afkSeconds % 60
            afkGui.MainFrame.TimeLabel.Text = string.format("%02d:%02d", minutes, seconds)
            if afkSeconds > 300 then
                afkGui.MainFrame.TimeLabel.TextColor3 = Color3.fromRGB(255, 120, 120)
            else
                afkGui.MainFrame.TimeLabel.TextColor3 = Color3.fromRGB(220, 240, 255)
            end
            task.wait(1)
        end
    end

    local function RunAntiAFK()
        while antiAfkEnabled do
            local currentTime = os.time()
            local now = tick()
            if currentTime - lastInputTime > 10 then
                if afkStartTime == 0 then
                    afkStartTime = currentTime
                    CreateAfkGui()
                    task.spawn(UpdateAfkTime)
                end
                if now - lastAfkAction >= AFK_ACTION_INTERVAL then
                    lastAfkAction = now
                    pcall(function()
                        local vu = game:GetService("VirtualUser")
                        vu:CaptureController()
                        vu:ClickButton2(Vector2.new(math.random(-5,5), math.random(-5,5)))
                    end)
                end
            else
                if afkStartTime > 0 then
                    afkStartTime = 0
                    if afkGui then afkGui:Destroy(); afkGui = nil end
                end
            end
            task.wait(1)
        end
        if afkGui then afkGui:Destroy(); afkGui = nil end
        afkStartTime = 0
    end

    UserInputService.InputBegan:Connect(function()
        lastInputTime = os.time()
    end)

    AutoFarmTab:AddToggle("AntiAFK", {
        Title = "Ativar Anti-AFK",
        Default = false,
        Callback = function(value)
            antiAfkEnabled = value
            if value then
                lastInputTime = os.time()
                afkThread = task.spawn(RunAntiAFK)
            else
                if afkThread then task.cancel(afkThread); afkThread = nil end
            end
        end
    })
end

-- Servidores
do
    ServersTab:AddSection("Encontrar Servidores")

    local findingServers = false

    local function showNotification(title, text)
        StarterGui:SetCore("SendNotification", {Title = title, Text = text, Duration = 5})
    end

    local function findAndJoinServers(playerCountFilter, customPlaceId)
        local gameId = customPlaceId or 18214855317
        local cursor = ""
        while findingServers do
            local success, response = pcall(function()
                local url = "https://games.roblox.com/v1/games/"..gameId.."/servers/Public?limit=100"
                if cursor ~= "" then url = url .. "&cursor="..cursor end
                return HttpService:JSONDecode(game:HttpGet(url))
            end)
            if success and response and response.data then
                for _, server in ipairs(response.data) do
                    local meetsCondition = false
                    if playerCountFilter == "low" then
                        meetsCondition = server.playing <= 1
                    elseif playerCountFilter == "medium" then
                        meetsCondition = server.playing >= 2 and server.playing <= 3
                    end
                    if meetsCondition and server.playing < server.maxPlayers then
                        showNotification("Servidor encontrado!", "Entrando em servidor com "..server.playing.." jogador(es)")
                        TeleportService:TeleportToPlaceInstance(gameId, server.id, LocalPlayer)
                        return
                    end
                end
                cursor = response.nextPageCursor or ""
            end
            showNotification("Procurando...", "Nenhum servidor adequado encontrado ainda")
            task.wait(10)
        end
    end

    local function toggleServerSearch(playerCountFilter, customPlaceId)
        findingServers = not findingServers
        if findingServers then
            showNotification("Iniciando busca", "Procurando servidores...")
            task.spawn(function() findAndJoinServers(playerCountFilter, customPlaceId) end)
        else
            showNotification("Busca cancelada", "Parou de procurar servidores")
        end
    end

    ServersTab:AddButton({
        Title = "Encontrar servidores OP (0-1 players)",
        Description = "Busca servidores com poucos jogadores",
        Callback = function() toggleServerSearch("low") end
    })

    ServersTab:AddButton({
        Title = "Encontrar servidores (2-3 players)",
        Description = "Busca servidores com alguns jogadores",
        Callback = function() toggleServerSearch("medium") end
    })
end

-- MISC
do
    MiscTab:AddSection("Configurações")
    
    MiscTab:AddToggle("DebugUI", {
        Title = "Debug Botões Mobile",
        Description = "Ative para ver o nome dos botões no console (F9) caso o Hitbox ative no botão errado",
        Default = false,
        Callback = function(val) debugUIEnabled = val end
    })

    MiscTab:AddSection("Tema do Painel")
    local themes = {"Dark", "Darker", "Light", "Aqua", "Amethyst", "Rose"}
    MiscTab:AddDropdown("ThemeSelect", {
        Title = "Selecionar Tema",
        Description = "Escolha o tema do painel",
        Values = themes,
        Default = "Darker",
        Multi = false,
        Callback = function(value) Fluent:SetTheme(value) end
    })
end

-- Créditos
do
    CreditTab:AddSection("Desenvolvido por Snow Team")
    CreditTab:AddButton({
        Title = "Snow Xit 12K",
        Description = "Entre no nosso servidor Discord!",
        Callback = function()
            local invite = "https://discord.gg/snowxit"
            if pcall(function()
                StarterGui:SetCore("OpenBrowserWindow", {Url = invite})
            end) then
                setclipboard(invite)
                StarterGui:SetCore("SendNotification", {
                    Title = "Convite copiado!",
                    Text = "O link do Discord foi copiado para sua área de transferência",
                    Duration = 5
                })
            end
        end
    })
end

Window:SelectTab(2)

Fluent:Notify({
    Title = "Snow Xit",
    Content = "Menu carregado com sucesso!",
    Duration = 5
})
