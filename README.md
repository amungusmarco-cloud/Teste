--!native
-- ========================================
-- ⚡ THUNDER HUB
-- Auto Farm + Kill Aura + Anti-Xit + Discord Error
-- Versão: 1.0
-- Criado por: THUNDER ⚡
-- ========================================

-- ========================================
-- CONFIGURAÇÕES DO CRIADOR
-- ========================================
local CREATOR = {
    Name = "THUNDER ⚡",
    Discord = "https://discord.gg/GTMEDtwmva",
    Version = "1.0",
    HubName = "⚡ THUNDER HUB"
}

-- ========================================
-- CARREGAR BIBLIOTECA FLUENT
-- ========================================
local Fluent: any
pcall(function()
    Fluent = loadstring(game:HttpGet("https://github.com/StyearX/Fluent-modded/releases/download/1.5.5/FluentPro"))()
end)

if not Fluent then
    warn("[THUNDER] Falha ao carregar Fluent")
    return
end

-- ========================================
-- SERVIÇOS
-- ========================================
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local ReplicatedStorage = game:GetService("ReplicatedStorage")
local VirtualInputManager = game:GetService("VirtualInputManager")
local TweenService = game:GetService("TweenService")
local TeleportService = game:GetService("TeleportService")
local UserInputService = game:GetService("UserInputService")
local Lighting = game:GetService("Lighting")

local LocalPlayer = Players.LocalPlayer
local Character = LocalPlayer.Character or LocalPlayer.CharacterAdded:Wait()

-- ========================================
-- CONFIGURAÇÕES DO HUB
-- ========================================
local CONFIG = {
    DiscordInvite = CREATOR.Discord,
    DiscordServerName = "Thunder Hub Community",
    AntiKick = true,
    AntiBan = true,
    AntiTeleport = true,
    AutoReconnect = true,
    KillAuraRange = 50,
    KillAuraDelay = 0.1,
    DungeonWaitTime = 2,
}

-- ========================================
-- VARIÁVEIS GLOBAIS
-- ========================================
_G.KillAura = false
_G.AntiDodge = false
_G.AutoRepeatDungeon = false
_G.SelectedDungeon = "Normal"
_G.CurrentTheme = "Thunder"
_G.KillAuraRange = CONFIG.KillAuraRange
_G.KillAuraDelay = CONFIG.KillAuraDelay

-- ========================================
-- LISTA DE DUNGEONS
-- ========================================
local DungeonList: {string} = {
    "Normal",
    "Hard", 
    "Nightmare",
    "Hell",
    "Tower",
    "Abyss"
}

-- ========================================
-- SISTEMA ANTI-XIT COMPLETO
-- ========================================

-- 1. BLOQUEAR KICK
local function BlockKicks()
    pcall(function()
        local oldKick = LocalPlayer.Kick
        LocalPlayer.Kick = function(self, ...)
            local args = {...}
            local reason = args[1] or "Desconhecido"
            ShowDiscordError("⚡ KICK DETECTADO", 
                "Você foi removido do servidor.\nMotivo: " .. tostring(reason) ..
                "\n\n🔹 Criado por: " .. CREATOR.Name
            )
            if CONFIG.AutoReconnect then
                task.wait(2)
                TeleportService:Teleport(game.PlaceId, LocalPlayer)
            end
            return nil
        end
        
        local mt = getrawmetatable(game)
        if mt then
            local oldNamecall = mt.__namecall
            mt.__namecall = function(self, ...)
                local method = getnamecallmethod()
                local args = {...}
                if method == "FireServer" and tostring(args[1]):lower():find("kick") then
                    ShowDiscordError("⚡ KICK REMOTO", 
                        "Tentativa de kick via Remote detectada!\n\n🔹 Criado por: " .. CREATOR.Name
                    )
                    return nil
                end
                if method == "InvokeServer" and tostring(args[1]):lower():find("kick") then
                    ShowDiscordError("⚡ KICK REMOTO", 
                        "Tentativa de kick via Remote detectada!\n\n🔹 Criado por: " .. CREATOR.Name
                    )
                    return nil
                end
                return oldNamecall(self, ...)
            end
        end
    end)
end

-- 2. BLOQUEAR BAN
local function BlockBan()
    pcall(function()
        LocalPlayer:GetPropertyChangedSignal("MembershipType"):Connect(function()
            if LocalPlayer.MembershipType == Enum.MembershipType.Banned then
                ShowDiscordError("⚡ CONTA BANIDA", 
                    "Sua conta foi banida!\nEntre em contato com o suporte.\n\n🔹 Criado por: " .. CREATOR.Name
                )
            end
        end)
    end)
end

-- 3. BLOQUEAR TELEPORTE FORÇADO
local function BlockForcedTeleport()
    pcall(function()
        local mt = getrawmetatable(game)
        if mt then
            local oldNamecall = mt.__namecall
            mt.__namecall = function(self, ...)
                local method = getnamecallmethod()
                local args = {...}
                if method == "FireServer" and tostring(args[1]):lower():find("teleport") then
                    if CONFIG.AntiTeleport then
                        ShowDiscordError("⚡ TELEPORTE BLOQUEADO", 
                            "Tentativa de teleporte forçado detectada!\n\n🔹 Criado por: " .. CREATOR.Name
                        )
                        return nil
                    end
                end
                return oldNamecall(self, ...)
            end
        end
    end)
end

-- 4. DETECTAR ATIVIDADE SUSPEITA
local function DetectSuspiciousActivity()
    pcall(function()
        LocalPlayer:GetPropertyChangedSignal("Parent"):Connect(function()
            if LocalPlayer.Parent == nil then
                ShowDiscordError("⚡ CONEXÃO PERDIDA", 
                    "Você foi desconectado do servidor!\n\n🔹 Criado por: " .. CREATOR.Name
                )
                if CONFIG.AutoReconnect then
                    task.wait(2)
                    TeleportService:Teleport(game.PlaceId, LocalPlayer)
                end
            end
        end)
    end)
end

-- 5. AUTO-RECONEXÃO
local function AutoReconnect()
    pcall(function()
        if CONFIG.AutoReconnect then
            LocalPlayer:GetPropertyChangedSignal("Parent"):Connect(function()
                if LocalPlayer.Parent == nil then
                    task.wait(2)
                    TeleportService:Teleport(game.PlaceId, LocalPlayer)
                end
            end)
        end
    end)
end

-- 6. MOSTRAR ERRO COM DISCORD + CRÉDITOS
local function ShowDiscordError(title: string, message: string)
    pcall(function()
        local screenGui = Instance.new("ScreenGui")
        screenGui.Name = "ThunderError"
        screenGui.ResetOnSpawn = false
        screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
        screenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")
        
        local backdrop = Instance.new("Frame")
        backdrop.Size = UDim2.new(1, 0, 1, 0)
        backdrop.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
        backdrop.BackgroundTransparency = 0.7
        backdrop.BorderSizePixel = 0
        backdrop.Parent = screenGui
        
        local frame = Instance.new("Frame")
        frame.Size = UDim2.new(0, 500, 0, 380)
        frame.Position = UDim2.new(0.5, -250, 0.5, -190)
        frame.BackgroundColor3 = Color3.fromRGB(10, 20, 40)
        frame.BackgroundTransparency = 0.05
        frame.BorderSizePixel = 0
        frame.Parent = screenGui
        
        local border = Instance.new("Frame")
        border.Size = UDim2.new(1, 0, 1, 0)
        border.Position = UDim2.new(0, 0, 0, 0)
        border.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
        border.BackgroundTransparency = 0.8
        border.BorderSizePixel = 0
        border.Parent = frame
        
        local borderCorner = Instance.new("UICorner")
        borderCorner.CornerRadius = UDim.new(0, 16)
        borderCorner.Parent = border
        
        local corner = Instance.new("UICorner")
        corner.CornerRadius = UDim.new(0, 16)
        corner.Parent = frame
        
        local titleLabel = Instance.new("TextLabel")
        titleLabel.Size = UDim2.new(1, 0, 0, 60)
        titleLabel.Position = UDim2.new(0, 0, 0, 10)
        titleLabel.BackgroundTransparency = 1
        titleLabel.Text = "⚡ " .. title
        titleLabel.TextColor3 = Color3.fromRGB(0, 191, 255)
        titleLabel.TextSize = 28
        titleLabel.Font = Enum.Font.SourceSansBold
        titleLabel.TextXAlignment = Enum.TextXAlignment.Center
        titleLabel.Parent = frame
        
        local divider = Instance.new("Frame")
        divider.Size = UDim2.new(0.8, 0, 0, 2)
        divider.Position = UDim2.new(0.1, 0, 0, 75)
        divider.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
        divider.BorderSizePixel = 0
        divider.Parent = frame
        
        local msgLabel = Instance.new("TextLabel")
        msgLabel.Size = UDim2.new(0.9, 0, 0, 110)
        msgLabel.Position = UDim2.new(0.05, 0, 0.25, 0)
        msgLabel.BackgroundTransparency = 1
        msgLabel.Text = message .. "\n\n🔹 Entre no Discord para resolver:"
        msgLabel.TextColor3 = Color3.fromRGB(200, 210, 230)
        msgLabel.TextSize = 16
        msgLabel.Font = Enum.Font.SourceSans
        msgLabel.TextWrapped = true
        msgLabel.TextXAlignment = Enum.TextXAlignment.Center
        msgLabel.TextYAlignment = Enum.TextYAlignment.Top
        msgLabel.Parent = frame
        
        local creditLabel = Instance.new("TextLabel")
        creditLabel.Size = UDim2.new(1, 0, 0, 30)
        creditLabel.Position = UDim2.new(0, 0, 0.9, 0)
        creditLabel.BackgroundTransparency = 1
        creditLabel.Text = "⚡ Criado por: " .. CREATOR.Name
        creditLabel.TextColor3 = Color3.fromRGB(0, 191, 255)
        creditLabel.TextSize = 14
        creditLabel.Font = Enum.Font.SourceSans
        creditLabel.TextXAlignment = Enum.TextXAlignment.Center
        creditLabel.Parent = frame
        
        local discordBtn = Instance.new("TextButton")
        discordBtn.Size = UDim2.new(0.7, 0, 0, 50)
        discordBtn.Position = UDim2.new(0.15, 0, 0.65, 0)
        discordBtn.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
        discordBtn.Text = "📱 ABRIR DISCORD"
        discordBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
        discordBtn.TextSize = 18
        discordBtn.Font = Enum.Font.SourceSansBold
        discordBtn.Parent = frame
        
        local discordCorner = Instance.new("UICorner")
        discordCorner.CornerRadius = UDim.new(0, 8)
        discordCorner.Parent = discordBtn
        
        discordBtn.MouseEnter:Connect(function()
            discordBtn.BackgroundColor3 = Color3.fromRGB(0, 150, 220)
        end)
        discordBtn.MouseLeave:Connect(function()
            discordBtn.BackgroundColor3 = Color3.fromRGB(0, 191, 255)
        end)
        
        local copyBtn = Instance.new("TextButton")
        copyBtn.Size = UDim2.new(0.3, 0, 0, 35)
        copyBtn.Position = UDim2.new(0.35, 0, 0.85, 0)
        copyBtn.BackgroundColor3 = Color3.fromRGB(30, 40, 60)
        copyBtn.Text = "📋 Copiar"
        copyBtn.TextColor3 = Color3.fromRGB(0, 191, 255)
        copyBtn.TextSize = 14
        copyBtn.Font = Enum.Font.SourceSans
        copyBtn.Parent = frame
        
        local copyCorner = Instance.new("UICorner")
        copyCorner.CornerRadius = UDim.new(0, 6)
        copyCorner.Parent = copyBtn
        
        local closeBtn = Instance.new("TextButton")
        closeBtn.Size = UDim2.new(0, 35, 0, 35)
        closeBtn.Position = UDim2.new(1, -45, 0, 5)
        closeBtn.BackgroundTransparency = 1
        closeBtn.Text = "✕"
        closeBtn.TextColor3 = Color3.fromRGB(200, 200, 210)
        closeBtn.TextSize = 22
        closeBtn.Font = Enum.Font.SourceSansBold
        closeBtn.Parent = frame
        
        discordBtn.MouseButton1Click:Connect(function()
            pcall(function()
                setclipboard(CONFIG.DiscordInvite)
                Fluent:Notify({
                    Title = CREATOR.HubName,
                    Content = "Convite copiado! " .. CONFIG.DiscordInvite,
                    Duration = 5
                })
            end)
        end)
        
        copyBtn.MouseButton1Click:Connect(function()
            pcall(function()
                setclipboard(CONFIG.DiscordInvite)
                Fluent:Notify({
                    Title = CREATOR.HubName,
                    Content = "📋 Convite copiado: " .. CONFIG.DiscordInvite,
                    Duration = 3
                })
            end)
        end)
        
        closeBtn.MouseButton1Click:Connect(function()
            screenGui:Destroy()
        end)
        
        frame.BackgroundTransparency = 0.05
        local tween = TweenService:Create(frame, TweenInfo.new(0.4, Enum.EasingStyle.Back), {
            Position = UDim2.new(0.5, -250, 0.5, -190)
        })
        tween:Play()
        
        task.delay(30, function()
            if screenGui and screenGui.Parent then
                screenGui:Destroy()
            end
        end)
    end)
end

-- ========================================
-- INICIAR ANTI-XIT
-- ========================================
task.spawn(function()
    pcall(function()
        BlockKicks()
        BlockBan()
        BlockForcedTeleport()
        DetectSuspiciousActivity()
        AutoReconnect()
        print("⚡ [THUNDER] Anti-Xit ativado! | Criado por: " .. CREATOR.Name)
    end)
end)

-- ========================================
-- FUNÇÕES DO JOGO
-- ========================================

-- Verificar se está na dungeon
local function IsInDungeon()
    local map = Workspace:FindFirstChild("Map")
    if map then
        for _, child in map:GetChildren() do
            if child.Name:find("Dungeon") or child.Name:find("Floor") then
                return true
            end
        end
    end
    return false
end

-- Obter alvos
local function GetTargets()
    local targets = {}
    local hrp = Character and Character:FindFirstChild("HumanoidRootPart")
    if not hrp then
        return targets
    end
    
    local pos = hrp.Position
    local range = _G.KillAuraRange or 50
    
    local enemiesFolder = Workspace:FindFirstChild("Enemies") 
        or Workspace:FindFirstChild("Mobs") 
        or Workspace:FindFirstChild("Monsters")
    
    if enemiesFolder then
        for _, mob in enemiesFolder:GetChildren() do
            if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                local root = mob:FindFirstChild("HumanoidRootPart") or mob:FindFirstChild("Torso")
                if root then
                    local dist = (root.Position - pos).Magnitude
                    if dist <= range then
                        table.insert(targets, {
                            Character = mob,
                            Humanoid = mob.Humanoid,
                            RootPart = root,
                            Distance = dist,
                            IsNPC = true
                        })
                    end
                end
            end
        end
    end
    
    table.sort(targets, function(a, b) return a.Distance < b.Distance end)
    return targets
end

-- Atacar alvo
local function AttackTarget(target)
    if not target or not target.Character then
        return false
    end
    if target.Humanoid and target.Humanoid.Health > 0 then
        target.Humanoid.Health = 0
        return true
    end
    return false
end

-- Kill Aura Loop
task.spawn(function()
    while task.wait(_G.KillAuraDelay or 0.1) do
        if not _G.KillAura then
            continue
        end
        pcall(function()
            local targets = GetTargets()
            for _, target in targets do
                if target.Humanoid and target.Humanoid.Health > 0 then
                    AttackTarget(target)
                    break
                end
            end
        end)
    end
end)

-- Anti-Dodge
task.spawn(function()
    while task.wait(0.05) do
        if not _G.AntiDodge or not _G.KillAura then
            continue
        end
        pcall(function()
            local targets = GetTargets()
            for _, target in targets do
                if target.RootPart then
                    if target.Character:FindFirstChild("Humanoid") then
                        local hum = target.Character.Humanoid
                        hum.WalkSpeed = 0
                        hum.JumpPower = 0
                    end
                    target.RootPart.Velocity = Vector3.zero
                    target.RootPart.RotVelocity = Vector3.zero
                end
            end
        end)
    end
end)

-- Auto Repeat Dungeon
local function StartDungeon()
    pcall(function()
        local startRemote = ReplicatedStorage:FindFirstChild("StartDungeon") 
            or ReplicatedStorage:FindFirstChild("DungeonStart")
            or ReplicatedStorage:FindFirstChild("Queue")
        
        if startRemote then
            if startRemote:IsA("RemoteEvent") then
                startRemote:FireServer(_G.SelectedDungeon)
            elseif startRemote:IsA("RemoteFunction") then
                startRemote:InvokeServer(_G.SelectedDungeon)
            end
        end
    end)
end

local function IsDungeonComplete()
    local enemies = Workspace:FindFirstChild("Enemies") or Workspace:FindFirstChild("Mobs")
    if enemies then
        local hasAlive = false
        for _, mob in enemies:GetChildren() do
            if mob:FindFirstChild("Humanoid") and mob.Humanoid.Health > 0 then
                if mob.Name:find("Boss") or mob.Name:find("King") or mob.Name:find("Lord") then
                    hasAlive = true
                    break
                end
            end
        end
        if not hasAlive then
            return true
        end
    end
    return false
end

task.spawn(function()
    while task.wait(0.5) do
        if not _G.AutoRepeatDungeon then
            continue
        end
        pcall(function()
            if not IsInDungeon() then
                StartDungeon()
                task.wait(CONFIG.DungeonWaitTime or 2)
            else
                if IsDungeonComplete() then
                    task.wait(2)
                    StartDungeon()
                    task.wait(CONFIG.DungeonWaitTime or 2)
                end
            end
        end)
    end
end)

-- ========================================
-- INTERFACE THUNDER HUB
-- ========================================

-- Registrar temas personalizados
pcall(function()
    Fluent:RegisterCustomTheme("Thunder", {
        Accent = Color3.fromRGB(0, 191, 255),
        AcrylicMain = Color3.fromRGB(10, 20, 40),
        Text = Color3.fromRGB(220, 235, 255),
        SubText = Color3.fromRGB(100, 180, 230),
        ToggleToggled = Color3.fromRGB(0, 191, 255),
    })
    
    Fluent:RegisterCustomTheme("DarkGold", {
        Accent = Color3.fromRGB(255, 215, 0),
        AcrylicMain = Color3.fromRGB(10, 10, 10),
        Text = Color3.fromRGB(230, 215, 180),
        SubText = Color3.fromRGB(180, 160, 100),
        ToggleToggled = Color3.fromRGB(255, 215, 0),
    })
end)

-- Criar Janela
local Window = Fluent:CreateWindow({
    Title = "⚡ THUNDER HUB",
    SubTitle = "Criado por: " .. CREATOR.Name,
    TabWidth = 130,
    Size = UDim2.fromOffset(520, 420),
    Acrylic = true,
    Theme = "Thunder",
    MinimizeKey = Enum.KeyCode.LeftControl,
})

-- ========================================
-- TAB PRINCIPAL
-- ========================================
local MainTab = Window:AddTab({ Title = "⚡ Main", Icon = "zap" })

MainTab:AddToggle("KillAura_Toggle", {
    Title = "⚡ Kill Aura",
    Description = "Ataca automaticamente os inimigos",
    Default = false,
    Callback = function(Value)
        _G.KillAura = Value
        Window:Notify({
            Title = CREATOR.HubName,
            Content = Value and "⚡ Kill Aura Ativada!" or "⚡ Kill Aura Desativada!",
            Duration = 2
      # Teste
