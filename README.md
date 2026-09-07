-- ========================================
-- ⚡ THUNDER HUB - DUNGEON QUEST REBORN
-- Versão: 2.0
-- Criado por: THUNDER ⚡
-- ========================================

-- ========================================
-- CONFIGURAÇÕES
-- ========================================
local CREATOR = "THUNDER ⚡"
local DISCORD = "https://discord.gg/SEUINVITE"
local VERSION = "2.0"

-- ========================================
-- SERVIÇOS
-- ========================================
local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local TeleportService = game:GetService("TeleportService")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

-- ========================================
-- VARIÁVEIS GLOBAIS
-- ========================================
_G.KillAura = false
_G.AntiDodge = false
_G.AutoRepeat = false
_G.AutoFarm = false
_G.SelectedDungeon = "Normal"
_G.KillAuraRange = 50
_G.KillAuraDelay = 0.1
_G.AutoHeal = true
_G.HealThreshold = 30

-- ========================================
-- LISTA DE DUNGEONS
-- ========================================
local DungeonList = {
    "Normal",
    "Hard", 
    "Nightmare",
    "Hell",
    "Tower",
    "Abyss"
}

-- ========================================
-- VERIFICAR SE É DUNGEON QUEST
-- ========================================
local function IsDungeonQuest()
    return game.PlaceId == 15590669150 or game.PlaceId == 4861900093
end

local function IsInDungeon()
    local map = Workspace:FindFirstChild("Map")
    if map then
        for _, child in map:GetChildren() do
            if child.Name:find("Dungeon") or child.Name:find("Floor") or child.Name:find("Stage") then
                return true
            end
        end
    end
    return false
end

local function IsDungeonComplete()
    local enemies = Workspace:FindFirstChild("Enemies") or Workspace:FindFirstChild("Mobs")
    if enemies then
        for _, mob in enemies:GetChildren() do
            if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                if mob.Name:find("Boss") or mob.Name:find("King") or mob.Name:find("Lord") or mob.Name:find("Demon") then
                    return false
                end
            end
        end
        return true
    end
    return false
end

-- ========================================
-- FUNÇÕES DE COMBATE
-- ========================================

local function GetTargets()
    local targets = {}
    local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
    if not hrp then return targets end
    
    local pos = hrp.Position
    local range = _G.KillAuraRange or 50
    
    local enemiesFolder = Workspace:FindFirstChild("Enemies") 
        or Workspace:FindFirstChild("Mobs") 
        or Workspace:FindFirstChild("Monsters")
    
    if enemiesFolder then
        for _, mob in enemiesFolder:GetChildren() do
            if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                local root = mob:FindFirstChild("HumanoidRootPart") or mob:FindFirstChild("Torso") or mob:FindFirstChild("UpperTorso")
                if root then
                    local dist = (root.Position - pos).Magnitude
                    if dist <= range then
                        table.insert(targets, {
                            Character = mob,
                            Humanoid = mob.Humanoid,
                            RootPart = root,
                            Distance = dist
                        })
                    end
                end
            end
        end
    end
    
    table.sort(targets, function(a, b) return a.Distance < b.Distance end)
    return targets
end

local function AttackTarget(target)
    if not target or not target.Character then return false end
    if target.Humanoid and target.Humanoid.Health > 0 then
        target.Humanoid.Health = 0
        return true
    end
    return false
end

local function UseSkill(key)
    local vim = game:GetService("VirtualInputManager")
    pcall(function()
        vim:SendKeyEvent(true, key, false, game)
        task.wait(0.05)
        vim:SendKeyEvent(false, key, false, game)
    end)
end

-- ========================================
-- LOOP: KILL AURA
-- ========================================
task.spawn(function()
    while task.wait(_G.KillAuraDelay or 0.1) do
        if not _G.KillAura then continue end
        pcall(function()
            local targets = GetTargets()
            if #targets > 0 then
                for _, target in targets do
                    if target.Humanoid and target.Humanoid.Health > 0 then
                        AttackTarget(target)
                        break
                    end
                end
            end
        end)
    end
end)

-- ========================================
-- LOOP: ANTI-DODGE
-- ========================================
task.spawn(function()
    while task.wait(0.05) do
        if not _G.AntiDodge or not _G.KillAura then continue end
        pcall(function()
            local targets = GetTargets()
            for _, target in targets do
                if target.RootPart then
                    if target.Character:FindFirstChild("Humanoid") then
                        target.Character.Humanoid.WalkSpeed = 0
                        target.Character.Humanoid.JumpPower = 0
                    end
                    target.RootPart.Velocity = Vector3.zero
                    target.RootPart.CFrame = target.RootPart.CFrame
                end
            end
        end)
    end
end)

-- ========================================
-- LOOP: AUTO HEAL
-- ========================================
task.spawn(function()
    while task.wait(0.5) do
        if not _G.AutoHeal then continue end
        pcall(function()
            local hum = Character and Character:FindFirstChild("Humanoid")
            if hum then
                local healthPercent = (hum.Health / hum.MaxHealth) * 100
                if healthPercent <= _G.HealThreshold then
                    UseSkill("Q")
                end
            end
        end)
    end
end)

-- ========================================
-- FUNÇÕES: DUNGEON
-- ========================================

local function StartDungeon()
    pcall(function()
        local remotes = {
            ReplicatedStorage:FindFirstChild("StartDungeon"),
            ReplicatedStorage:FindFirstChild("DungeonStart"),
            ReplicatedStorage:FindFirstChild("Queue"),
            ReplicatedStorage:FindFirstChild("JoinDungeon"),
            ReplicatedStorage:FindFirstChild("EnterDungeon")
        }
        
        for _, remote in pairs(remotes) do
            if remote then
                if remote:IsA("RemoteEvent") then
                    remote:FireServer(_G.SelectedDungeon)
                elseif remote:IsA("RemoteFunction") then
                    remote:InvokeServer(_G.SelectedDungeon)
                end
                break
            end
        end
        
        local gui = LocalPlayer.PlayerGui:FindFirstChild("DungeonGUI") 
            or LocalPlayer.PlayerGui:FindFirstChild("Main")
        if gui then
            local startButton = gui:FindFirstChild("StartButton") 
                or gui:FindFirstChild("QueueButton")
                or gui:FindFirstChild("PlayButton")
            if startButton and startButton:IsA("TextButton") then
                startButton:FireServer()
            end
        end
    end)
end

local function LeaveDungeon()
    pcall(function()
        local remotes = {
            ReplicatedStorage:FindFirstChild("LeaveDungeon"),
            ReplicatedStorage:FindFirstChild("ExitDungeon"),
            ReplicatedStorage:FindFirstChild("Leave")
        }
        
        for _, remote in pairs(remotes) do
            if remote then
                if remote:IsA("RemoteEvent") then
                    remote:FireServer()
                elseif remote:IsA("RemoteFunction") then
                    remote:InvokeServer()
                end
                break
            end
        end
    end)
end

-- ========================================
-- LOOP: AUTO REPEAT DUNGEON
-- ========================================
task.spawn(function()
    while task.wait(0.5) do
        if not _G.AutoRepeat then continue end
        pcall(function()
            local inDungeon = IsInDungeon()
            
            if not inDungeon then
                StartDungeon()
                task.wait(2)
            else
                if IsDungeonComplete() then
                    task.wait(2)
                    LeaveDungeon()
                    task.wait(2)
                    StartDungeon()
                    task.wait(2)
                end
            end
        end)
    end
end)

-- ========================================
-- LOOP: AUTO FARM
-- ========================================
task.spawn(function()
    while task.wait(0.2) do
        if not _G.AutoFarm then continue end
        
        _G.KillAura = true
        _G.AutoRepeat = true
        
        pcall(function()
            if IsInDungeon() then
                local targets = GetTargets()
                for _, target in targets do
                    if target.Humanoid and target.Humanoid.Health > 0 then
                        AttackTarget(target)
                        break
                    end
                end
            end
        end)
    end
end)

-- ========================================
-- MOSTRAR ERRO COM DISCORD
-- ========================================
local function ShowDiscordError(title, message)
    pcall(function()
        local screenGui = Instance.new("ScreenGui")
        screenGui.Name = "ThunderError"
        screenGui.ResetOnSpawn = false
        screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
        
        local backdrop = Instance.new("Frame")
        backdrop.Size = UDim2.new(1, 0, 1, 0)
        backdrop.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        backdrop.BackgroundTransparency = 0.6
        backdrop.BorderSizePixel = 0
        backdrop.Parent = screenGui
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(0, 400, 0, 280)
        frame.Position = UDim2.new(0.5, -200, 0.5, -140)
        frame.BackgroundColor3 = Color3.fromRGB(10, 20, 40)
        frame.BackgroundTransparency = 0.1
        frame.BorderSizePixel = 0
        frame.Parent = screenGui
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 12)
        corner.Parent = frame
        
        local titleLabel = Instance.new("TextLabel")
        titleLabel.Size = UDim2.new(1, 0, 0, 50)
        titleLabel.Position = UDim2.new(0, 0, 0, 10)
        titleLabel.BackgroundTransparency = 1
        titleLabel.Text = "⚡ " .. title
        titleLabel.TextColor3 = Color3.fromRGB(0, 191, 255)
        titleLabel.TextSize = 22
        titleLabel.Font = Enum.Font.SourceSansBold
        titleLabel.TextXAlignment = Enum.TextXAlignment.Center
        titleLabel.Parent = frame
        
        local msgLabel = Instance.new("TextLabel")
        msgLabel.Size = UDim2.new(0.9, 0, 0, 100)
        msgLabel.Position = UDim2.new(0.05, 0, 0.2, 0)
        msgLabel.BackgroundTransparency = 1
        msgLabel.Text = message .. "\n\n📱 Discord: " .. DISCORD
        msgLabel.TextColor3 = Color3.fromRGB(200, 210, 230)
        msgLabel.TextSize = 14
        msgLabel.Font = Enum.Font.SourceSans
        msgLabel.TextWrapped = true
        msgLabel.TextXAlignment = Enum.TextXAlignment.Center
        msgLabel.Parent = frame
        
        local copyBtn = Instance.new("TextButton")
        copyBtn.Size = UDim2.new(0.6, 0, 0, 40)
        copyBtn.Position = UDim2.new(0.2, 0, 0.6, 0)
        copyBtn.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
        copyBtn.Text = "📋 COPIAR DISCORD"
        copyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        copyBtn.TextSize = 16
        copyBtn.Font = Enum.Font.SourceSansBold
        copyBtn.Parent = frame
        
        local copyCorner = Instance.new("UICorner")
        copyCorner.CornerRadius = UDim.new(0, 8)
        copyCorner.Parent = copyBtn
        
        local closeBtn = Instance.new("TextButton")
        closeBtn.Size = UDim2.new(0, 30, 0, 30)
        closeBtn.Position = UDim2.new(1, -35, 0, 5)
        closeBtn.BackgroundTransparency = 1
        closeBtn.Text = "✕"
        closeBtn.TextColor3 = Color3.fromRGB(200, 200, 210)
        closeBtn.TextSize = 18
        closeBtn.Font = Enum.Font.SourceSansBold
        closeBtn.Parent = frame
        
        copyBtn.MouseButton1Click:Connect(function()
            pcall(function()
                if setclipboard then
                    setclipboard(DISCORD)
                end
            end)
        end)
        
        closeBtn.MouseButton1Click:Connect(function()
            screenGui:Destroy()
        end)
        
        local tween = TweenService:Create(frame, TweenInfo.new(0.3, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -200, 0.5, -140)
        })
        tween:Play()
        
        task.delay(30, function()
            if screenGui and screenGui.Parent then
                screenGui:Destroy()
            end
        end)
    end)
end)

-- ========================================
-- INTERFACE
-- ========================================
local function CreateUI()
    if _G.ThunderUI and _G.ThunderUI.Parent then
        _G.ThunderUI:Destroy()
    end
    
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "ThunderHub"
    screenGui.ResetOnSpawn = false
    screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
    screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
    _G.ThunderUI = screenGui
    
    local mainFrame = Instance.new("Frame")
    mainFrame.Size = UDim2.new(0, 380, 0, 520)
    mainFrame.Position = UDim2.new(0.5, -190, 0.5, -260)
    mainFrame.BackgroundColor3 = Color3.fromRGB(8, 16, 35)
    mainFrame.BackgroundTransparency = 0.05
    mainFrame.BorderSizePixel = 0
    mainFrame.Parent = screenGui
    
    local corner = Instance.new("UICorner")
    corner.CornerRadius = UDim.new(0, 14)
    corner.Parent = mainFrame
    
    local border = Instance.new("Frame")
    border.Size = UDim2.new(1, 0, 1, 0)
    border.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
    border.BackgroundTransparency = 0.6
    border.BorderSizePixel = 0
    border.Parent = mainFrame
    
    local borderCorner = Instance.new("UICorner")
    borderCorner.CornerRadius = UDim.new(0, 14)
    borderCorner.Parent = border
    
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, 0, 0, 50)
    title.Position = UDim2.new(0, 0, 0, 5)
    title.BackgroundTransparency = 1
    title.Text = "⚡ THUNDER HUB"
    title.TextColor3 = Color3.fromRGB(0, 191, 255)
    title.TextSize = 26
    title.Font = Enum.Font.SourceSansBold
    title.TextXAlignment = Enum.TextXAlignment.Center
    title.Parent = mainFrame
    
    local versionLabel = Instance.new("TextLabel")
    versionLabel.Size = UDim2.new(0, 60, 0, 20)
    versionLabel.Position = UDim2.new(1, -65, 0, 10)
    versionLabel.BackgroundTransparency = 1
    versionLabel.Text = "v" .. VERSION
    versionLabel.TextColor3 = Color3.fromRGB(100, 180, 230)
    versionLabel.TextSize = 12
    versionLabel.Font = Enum.Font.SourceSans
    versionLabel.TextXAlignment = Enum.TextXAlignment.Right
    versionLabel.Parent = mainFrame
    
    local subTitle = Instance.new("TextLabel")
    subTitle.Size = UDim2.new(1, 0, 0, 20)
    subTitle.Position = UDim2.new(0, 0, 0, 50)
    subTitle.BackgroundTransparency = 1
    subTitle.Text = "Criado por: " .. CREATOR
    subTitle.TextColor3 = Color3.fromRGB(130, 180, 230)
    subTitle.TextSize = 13
    subTitle.Font = Enum.Font.SourceSans
    subTitle.TextXAlignment = Enum.TextXAlignment.Center
    subTitle.Parent = mainFrame
    
    local divider = Instance.new("Frame")
    divider.Size = UDim2.new(0.9, 0, 0, 1)
    divider.Position = UDim2.new(0.05, 0, 0, 75)
    divider.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
    divider.BackgroundTransparency = 0.5
    divider.BorderSizePixel = 0
    divider.Parent = mainFrame
    
    local function CreateToggle(text, y, varName, color)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.85, 0, 0, 38)
        btn.Position = UDim2.new(0.075, 0, y/520, 0)
        btn.BackgroundColor3 = Color3.fromRGB(35, 40, 55)
        btn.Text = text .. " ❌ OFF"
        btn.TextColor3 = Color3.fromRGB(180, 190, 210)
        btn.TextSize = 15
        btn.Font = Enum.Font.SourceSansBold
        btn.Parent = mainFrame
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 6)
        btnCorner.Parent = btn
        
        if _G[varName] then
            btn.Text = text .. " ✅ ON"
            btn.BackgroundColor3 = color or Color3.fromRGB(0, 191, 255)
            btn.TextColor3 = Color3.fromRGB(255, 255, 255)
        end
        
        btn.MouseButton1Click:Connect(function()
            _G[varName] = not _G[varName]
            if _G[varName] then
                btn.Text = text .. " ✅ ON"
                btn.BackgroundColor3 = color or Color3.fromRGB(0, 191, 255)
                btn.TextColor3 = Color3.fromRGB(255, 255, 255)
            else
                btn.Text = text .. " ❌ OFF"
                btn.BackgroundColor3 = Color3.fromRGB(35, 40, 55)
                btn.TextColor3 = Color3.fromRGB(180, 190, 210)
            end
        end)
        
        return btn
    end
    
    local y = 85
    local spacing = 44
    
    CreateToggle("⚡ Kill Aura", y, "KillAura")
    y = y + spacing
    
    CreateToggle("🛡️ Anti-Dodge", y, "AntiDodge")
    y = y + spacing
    
    CreateToggle("🔄 Auto Repeat", y, "AutoRepeat")
    y = y + spacing
    
    CreateToggle("⭐ Auto Farm", y, "AutoFarm", Color3.fromRGB(255, 215, 0))
    y = y + spacing
    
    local sectionLabel = Instance.new("TextLabel")
    sectionLabel.Size = UDim2.new(0.85, 0, 0, 25)
    sectionLabel.Position = UDim2.new(0.075, 0, y/520, 0)
    sectionLabel.BackgroundTransparency = 1
    sectionLabel.Text = "🏰 SELECIONAR DUNGEON"
    sectionLabel.TextColor3 = Color3.fromRGB(0, 191, 255)
    sectionLabel.TextSize = 14
    sectionLabel.Font = Enum.Font.SourceSansBold
    sectionLabel.TextXAlignment = Enum.TextXAlignment.Center
    sectionLabel.Parent = mainFrame
    
    y = y + 30
    
    local dungeonLabel = Instance.new("TextLabel")
    dungeonLabel.Size = UDim2.new(0.85, 0, 0, 22)
    dungeonLabel.Position = UDim2.new(0.075, 0, y/520, 0)
    dungeonLabel.BackgroundTransparency = 1
    dungeonLabel.Text = "Atual: " .. _G.SelectedDungeon
    dungeonLabel.TextColor3 = Color3.fromRGB(200, 210, 230)
    dungeonLabel.TextSize = 13
    dungeonLabel.Font = Enum.Font.SourceSans
    dungeonLabel.TextXAlignment = Enum.TextXAlignment.Center
    dungeonLabel.Parent = mainFrame
    
    y = y + 27
    
    local function CreateDungeonBtn(text, xPos, yPos)
        local btn = Instance.new("TextButton")
        btn.Size = UDim2.new(0.25, 0, 0, 28)
        btn.Position = UDim2.new(xPos, 0, yPos/520, 0)
        btn.BackgroundColor3 = Color3.fromRGB(25, 40, 65)
        btn.Text = text
        btn.TextColor3 = Color3.fromRGB(180, 200, 220)
        btn.TextSize = 11
        btn.Font = Enum.Font.SourceSansBold
        btn.Parent = mainFrame
        
        local btnCorner = Instance.new("UICorner")
        btnCorner.CornerRadius = UDim.new(0, 4)
        btnCorner.Parent = btn
        
        if text == _G.SelectedDungeon then
            btn.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
            btn
