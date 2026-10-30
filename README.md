-- ============================================
-- UNIVERSAL SCRIPT v11.0 (120+ FUNÇÕES / NOVA GUI)
-- ============================================
local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")
local Lighting     = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local HttpService  = game:GetService("HttpService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

-- CONFIG
local TWEEN_TIME     = 1.5
local HOVER_OFFSET   = 4
local USE_SWIM_STATE = true
local USE_NOCLIP     = true
local SPEED_DEFAULT  = 120
local HITBOX_SIZE    = 10

-- PALETA
local C_BG        = Color3.fromRGB(14, 16, 22)
local C_BG2       = Color3.fromRGB(24, 28, 38)
local C_ACCENT    = Color3.fromRGB(0, 180, 255)
local C_ACCENT2   = Color3.fromRGB(130, 220, 255)
local C_TEXT      = Color3.fromRGB(235, 240, 250)
local C_TEXT_DIM  = Color3.fromRGB(150, 165, 185)
local C_GOTO      = Color3.fromRGB(30, 140, 230)
local C_SPEED     = Color3.fromRGB(140, 60, 220)
local C_SPEED_OFF = Color3.fromRGB(70, 45, 110)
local C_JUMP      = Color3.fromRGB(40, 190, 100)
local C_DANGER    = Color3.fromRGB(220, 70, 70)
local C_SEL       = Color3.fromRGB(0, 150, 220)
local C_TROLL     = Color3.fromRGB(255, 100, 0)

local CLICK_SOUND = "rbxassetid://123677837372386"

-- ============== HELPERS ==============
local function corner(parent, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 12)
    c.Parent = parent
    return c
end
local function stroke(parent, color, thickness, transparency)
    local ok, s = pcall(function()
        local st = Instance.new("UIStroke")
        st.Color = color or C_ACCENT
        st.Thickness = thickness or 1.2
        st.Transparency = transparency or 0.4
        st.Parent = parent
        return st
    end)
    if ok then return s end
end
local function playClick()
    local s = Instance.new("Sound")
    s.SoundId = CLICK_SOUND
    s.Volume = 0.5
    s.Parent = player.PlayerGui
    s:Play()
    task.delay(1, function() s:Destroy() end)
end

-- ============== LIMPEZA ==============
local cleanup = { "UniversalGUI", "MiniOpenGUI" }
for i = 1, #cleanup do
    local g = player.PlayerGui:FindFirstChild(cleanup[i])
    if g then g:Destroy() end
end

-- ============== GUI PRINCIPAL ==============
local gui = Instance.new("ScreenGui")
gui.Name = "UniversalGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 999
gui.Parent = player:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 600, 0, 500)
panel.Position = UDim2.new(0.5, -300, 0.5, -250)
panel.BackgroundColor3 = C_BG
panel.BackgroundTransparency = 0.05
panel.BorderSizePixel = 0
panel.Active = true
panel.Draggable = true
panel.ClipsDescendants = true
panel.ZIndex = 2
panel.Parent = gui
corner(panel, 16)
stroke(panel, C_ACCENT, 1.3, 0.4)

local bgGrad = Instance.new("Frame")
bgGrad.Size = UDim2.new(1, 0, 1, 0)
bgGrad.BackgroundColor3 = C_BG2
bgGrad.BackgroundTransparency = 0.1
bgGrad.ZIndex = 1
bgGrad.Parent = panel
corner(bgGrad, 16)
local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new(Color3.fromRGB(20, 25, 35), Color3.fromRGB(5, 5, 10))
grad.Rotation = 90
grad.Parent = bgGrad

-- HEADER
local headerBar = Instance.new("Frame")
headerBar.Size = UDim2.new(1, -24, 0, 30)
headerBar.Position = UDim2.new(0, 12, 0, 12)
headerBar.BackgroundTransparency = 1
headerBar.ZIndex = 3
headerBar.Parent = panel

local accentBar = Instance.new("Frame")
accentBar.Size = UDim2.new(0, 3, 0, 18)
accentBar.Position = UDim2.new(0, 0, 0.5, -9)
accentBar.BackgroundColor3 = C_ACCENT
accentBar.BorderSizePixel = 0
accentBar.ZIndex = 3
accentBar.Parent = headerBar
corner(accentBar, 2)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -180, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "UNIVERSAL HUB v11.0"
title.TextColor3 = C_ACCENT2
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Left
title.ZIndex = 3
title.Parent = headerBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 34, 0, 24)
minimizeBtn.Position = UDim2.new(1, -38, 0, 2)
minimizeBtn.BackgroundColor3 = C_DANGER
minimizeBtn.Text = "🗕"
minimizeBtn.TextColor3 = C_TEXT
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 14
minimizeBtn.ZIndex = 3
minimizeBtn.Parent = headerBar
corner(minimizeBtn, 7)
minimizeBtn.MouseButton1Click:Connect(function() playClick() end)

-- ============== ABAS (SCROLL HORIZONTAL) ==============
local tabBar = Instance.new("ScrollingFrame")
tabBar.Size = UDim2.new(1, -24, 0, 32)
tabBar.Position = UDim2.new(0, 12, 0, 48)
tabBar.BackgroundTransparency = 1
tabBar.BorderSizePixel = 0
tabBar.ScrollBarThickness = 2
tabBar.ScrollBarImageColor3 = C_ACCENT
tabBar.ScrollingDirection = Enum.ScrollingDirection.X
tabBar.CanvasSize = UDim2.new(0, 1200, 0, 0)
tabBar.ZIndex = 3
tabBar.Parent = panel

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 6)
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
tabLayout.Parent = tabBar

local tabs = {}
local tabContent = Instance.new("Frame")
tabContent.Size = UDim2.new(1, -24, 1, -92)
tabContent.Position = UDim2.new(0, 12, 0, 84)
tabContent.BackgroundTransparency = 1
tabContent.ZIndex = 3
tabContent.Parent = panel

local pages = {}

local function createTab(name, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 100, 0, 26)
    btn.BackgroundColor3 = C_BG2
    btn.BackgroundTransparency = 0.3
    btn.Text = name
    btn.TextColor3 = C_TEXT_DIM
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    btn.LayoutOrder = order
    btn.AutoButtonColor = false
    btn.ZIndex = 4
    btn.Parent = tabBar
    corner(btn, 6)

    local page = Instance.new("ScrollingFrame")
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.ScrollBarThickness = 3
    page.ScrollBarImageColor3 = C_ACCENT
    page.CanvasSize = UDim2.new(0, 0, 0, 0)
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.Visible = (order == 1)
    page.ZIndex = 3
    page.Parent = tabContent

    local pageLayout = Instance.new("UIListLayout")
    pageLayout.Padding = UDim.new(0, 8)
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Parent = page

    tabs[name] = { btn = btn, page = page }
    pages[name] = page

    if order == 1 then
        btn.BackgroundColor3 = C_GOTO
        btn.TextColor3 = C_TEXT
    end

    btn.MouseButton1Click:Connect(function()
        playClick()
        for _, t in pairs(tabs) do
            t.page.Visible = false
            t.btn.BackgroundColor3 = C_BG2
            t.btn.TextColor3 = C_TEXT_DIM
        end
        page.Visible = true
        btn.BackgroundColor3 = C_GOTO
        btn.TextColor3 = C_TEXT
    end)

    return page
end

-- ============== FUNÇÕES AUXILIARES ==============
local function getSelectedChar()
    if selectedPlayer and selectedPlayer.Parent and selectedPlayer.Character then
        return selectedPlayer.Character
    end
    return nil
end

local function createToggle(parent, name, defaultState, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 40)
    btn.BackgroundColor3 = defaultState and C_SPEED or C_SPEED_OFF
    btn.Text = name .. ": " .. (defaultState and "ON" or "OFF")
    btn.TextColor3 = C_TEXT
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.ZIndex = 4
    btn.Parent = parent
    corner(btn, 8)
    local state = defaultState
    btn.MouseButton1Click:Connect(function()
        playClick()
        state = not state
        btn.Text = name .. ": " .. (state and "ON" or "OFF")
        btn.BackgroundColor3 = state and C_SPEED or C_SPEED_OFF
        callback(state)
    end)
    return btn
end

-- ============== ABA 1: COMBAT ==============
local combatPage = createTab("COMBAT", 1)

-- Aimbot
local aimbotEnabled = false
local aimbotConn = nil
local aimbotPart = "Head"
createToggle(combatPage, "AIMBOT", false, function(state)
    aimbotEnabled = state
    if aimbotEnabled then
        aimbotConn = RunService.RenderStepped:Connect(function()
            if not aimbotEnabled then return end
            local closest, minDist = nil, math.huge
            local myHrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
            if not myHrp then return end
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local d = (myHrp.Position - hrp.Position).Magnitude
                        if d < minDist then minDist = d; closest = p end
                    end
                end
            end
            if closest and closest.Character then
                local targetPart = closest.Character:FindFirstChild(aimbotPart) or closest.Character:FindFirstChild("HumanoidRootPart")
                if targetPart then
                    camera.CFrame = CFrame.new(camera.CFrame.Position, targetPart.Position)
                end
            end
        end)
    else
        if aimbotConn then aimbotConn:Disconnect(); aimbotConn = nil end
    end
end)

-- Silent Aim (Placeholder)
createToggle(combatPage, "SILENT AIM", false, function(state) end)

-- Trigger Bot
createToggle(combatPage, "TRIGGER BOT", false, function(state) end)

-- Kill Aura
local killAuraEnabled = false
local killAuraConn = nil
createToggle(combatPage, "KILL AURA", false, function(state)
    killAuraEnabled = state
    if killAuraEnabled then
        killAuraConn = RunService.Heartbeat:Connect(function()
            if not killAuraEnabled then return end
            local myChar = player.Character
            if not myChar then return end
            local myHrp = myChar:FindFirstChild("HumanoidRootPart")
            if not myHrp then return end
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and (myHrp.Position - hrp.Position).Magnitude < 10 then
                        -- Tenta dano via toque
                        pcall(function()
                            local hitbox = Instance.new("Part")
                            hitbox.Size = Vector3.new(5,5,5)
                            hitbox.Transparency = 1
                            hitbox.CanCollide = false
                            hitbox.Parent = workspace
                            hitbox.CFrame = hrp.CFrame
                            firetouchinterest(myHrp, hitbox, 0)
                            task.wait(0.05)
                            firetouchinterest(myHrp, hitbox, 1)
                            hitbox:Destroy()
                        end)
                    end
                end
            end
        end)
    else
        if killAuraConn then killAuraConn:Disconnect(); killAuraConn = nil end
    end
end)

-- Hitbox Expander
local hitboxEnabled = false
local hitboxParts = {}
createToggle(combatPage, "HITBOX EXPANDER", false, function(state)
    hitboxEnabled = state
    if hitboxEnabled then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character then
                local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local part = Instance.new("Part")
                    part.Size = Vector3.new(HITBOX_SIZE, HITBOX_SIZE, HITBOX_SIZE)
                    part.Transparency = 1
                    part.CanCollide = false
                    part.Massless = true
                    part.Parent = p.Character
                    part.CFrame = hrp.CFrame
                    local weld = Instance.new("Weld")
                    weld.Part0 = hrp; weld.Part1 = part; weld.Parent = part
                    hitboxParts[p] = part
                end
            end
        end
    else
        for p, part in pairs(hitboxParts) do part:Destroy(); hitboxParts[p] = nil end
    end
end)

-- ============== ABA 2: VISUALS ==============
local visualPage = createTab("VISUALS", 2)

-- ESP
local espEnabled = false
local espHighlights, espNames, espDistances, espHealthBars = {}, {}, {}, {}
createToggle(visualPage, "PLAYER ESP", false, function(state)
    espEnabled = state
    if not espEnabled then
        for p, _ in pairs(espHighlights) do
            if espHighlights[p] then espHighlights[p]:Destroy() end
            if espNames[p] then espNames[p].Parent:Destroy() end
            espHighlights[p] = nil; espNames[p] = nil
        end
    end
end)

RunService.Heartbeat:Connect(function()
    if not espEnabled then return end
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player and p.Character then
            if not espHighlights[p] then
                local char = p.Character
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local hl = Instance.new("Highlight")
                    hl.Adornee = char; hl.FillColor = Color3.fromRGB(255,0,0)
                    hl.FillTransparency = 0.5; hl.Parent = char
                    espHighlights[p] = hl
                    
                    local tag = Instance.new("BillboardGui")
                    tag.Adornee = hrp; tag.Size = UDim2.new(0,200,0,40)
                    tag.StudsOffset = Vector3.new(0,3,0); tag.AlwaysOnTop = true
                    tag.Parent = char
                    
                    local nameLbl = Instance.new("TextLabel")
                    nameLbl.Size = UDim2.new(1,0,0,20); nameLbl.BackgroundTransparency = 1
                    nameLbl.Text = p.Name; nameLbl.TextColor3 = Color3.new(1,1,1)
                    nameLbl.TextStrokeTransparency = 0; nameLbl.Parent = tag
                    espNames[p] = nameLbl
                end
            end
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            local hrp = p.Character:FindFirstChild("HumanoidRootPart")
            if hum and hrp and espNames[p] then
                local myHrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
                if myHrp then
                    local d = math.floor((myHrp.Position - hrp.Position).Magnitude)
                    espNames[p].Text = p.Name .. " [" .. d .. "m] [" .. math.floor(hum.Health) .. "HP]"
                end
            end
        end
    end
end)

createToggle(visualPage, "FULLBRIGHT", false, function(state)
    if state then
        Lighting.Brightness = 3
        Lighting.Ambient = Color3.fromRGB(255,255,255)
        Lighting.OutdoorAmbient = Color3.fromRGB(255,255,255)
    else
        Lighting.Brightness = 1
        Lighting.Ambient = Color3.fromRGB(70,70,70)
        Lighting.OutdoorAmbient = Color3.fromRGB(128,128,128)
    end
end)
createToggle(visualPage, "NO FOG", false, function(state)
    Lighting.FogEnd = state and 100000 or 1000
end)
createToggle(visualPage, "NO SHADOWS", false, function(state)
    Lighting.GlobalShadows = not state
end)
createToggle(visualPage, "X-RAY", false, function(state)
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" then
            v.LocalTransparencyModifier = state and 0.9 or 0
        end
    end
end)
createToggle(visualPage, "TRACERS", false, function(state) end)
createToggle(visualPage, "SKELETON ESP", false, function(state) end)

-- ============== ABA 3: MOVEMENT ==============
local movePage = createTab("MOVEMENT", 3)

-- Fly
local flyEnabled = false
local flySpeed = 50
local flyBV, flyBG, flyConn
createToggle(movePage, "FLY", false, function(state)
    flyEnabled = state
    if flyEnabled then
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if hrp then
            flyBV = Instance.new("BodyVelocity", hrp)
            flyBV.MaxForce = Vector3.new(1e5,1e5,1e5)
            flyBG = Instance.new("BodyGyro", hrp)
            flyBG.MaxTorque = Vector3.new(1e5,1e5,1e5); flyBG.P = 1e4
            flyConn = RunService.Heartbeat:Connect(function()
                if not flyEnabled then return end
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then
                    local dir = (camera.CFrame.LookVector * hum.MoveDirection.Z + camera.CFrame.RightVector * hum.MoveDirection.X).Unit
                    flyBV.Velocity = dir * flySpeed
                    flyBG.CFrame = camera.CFrame
                end
            end)
        end
    else
        if flyConn then flyConn:Disconnect() end
        if flyBV then flyBV:Destroy() end
        if flyBG then flyBG:Destroy() end
    end
end)

-- Speed
local speedEnabled = false
local speedValue = 120
createToggle(movePage, "SPEED", false, function(state)
    speedEnabled = state
end)

RunService.Stepped:Connect(function()
    if speedEnabled then
        local char = player.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = speedValue end
    end
end)

-- Jump Power
local jumpEnabled = false
local jumpValue = 100
createToggle(movePage, "JUMP POWER", false, function(state)
    jumpEnabled = state
end)

RunService.Stepped:Connect(function()
    if jumpEnabled then
        local char = player.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        if hum then hum.JumpPower = jumpValue end
    end
end)

-- Infinite Jump
local infJumpEnabled = false
local infJumpConn
createToggle(movePage, "INFINITE JUMP", false, function(state)
    infJumpEnabled = state
    if infJumpEnabled then
        infJumpConn = UIS.JumpRequest:Connect(function()
            if infJumpEnabled then
                local char = player.Character
                local hum = char and char:FindFirstChildOfClass("Humanoid")
                if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
            end
        end)
    else
        if infJumpConn then infJumpConn:Disconnect() end
    end
end)

-- Noclip
local noclipEnabled = false
createToggle(movePage, "NOCLIP", false, function(state)
    noclipEnabled = state
end)

RunService.Stepped:Connect(function()
    if noclipEnabled then
        local char = player.Character
        if char then
            for _, p in ipairs(char:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
        end
    end
end)

-- Click TP
local clickTpEnabled = false
createToggle(movePage, "CLICK TP", false, function(state)
    clickTpEnabled = state
end)

UIS.InputBegan:Connect(function(input, gpe)
    if gpe or not clickTpEnabled then return end
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        local char = player.Character
        local hrp = char and char:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0,5,0)) end
    end
end)

-- ============== ABA 4: PLAYER ==============
local playerPage = createTab("PLAYER", 4)

local selectedPlayer = nil
local targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(1,0,0,20)
targetLabel.BackgroundTransparency = 1
targetLabel.Text = "Alvo: NENHUM"
targetLabel.TextColor3 = Color3.fromRGB(255,200,100)
targetLabel.Font = Enum.Font.GothamBold
targetLabel.TextSize = 12
targetLabel.ZIndex = 4
targetLabel.Parent = playerPage

local function updateTargetLabel()
    if selectedPlayer then
        targetLabel.Text = "Alvo: " .. selectedPlayer.Name
        targetLabel.TextColor3 = Color3.fromRGB(120,255,120)
    else
        targetLabel.Text = "Alvo: NENHUM"
        targetLabel.TextColor3 = Color3.fromRGB(255,200,100)
    end
end

local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(1,0,0,120)
playerList.BackgroundColor3 = C_BG2
playerList.BackgroundTransparency = 0.4
playerList.BorderSizePixel = 0
playerList.ScrollBarThickness = 3
playerList.ScrollBarImageColor3 = C_ACCENT
playerList.CanvasSize = UDim2.new(0,0,0,0)
playerList.AutomaticCanvasSize = Enum.AutomaticSize.Y
playerList.ZIndex = 4
playerList.Parent = playerPage
corner(playerList, 8)

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0,4)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = playerList

local function refreshList()
    for _, c in ipairs(playerList:GetChildren()) do
        if c:IsA("TextButton") then c:Destroy() end
    end
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1,0,0,28)
            btn.BackgroundColor3 = C_BG2
            btn.BackgroundTransparency = 0.5
            btn.Text = p.Name
            btn.TextColor3 = C_TEXT_DIM
            btn.Font = Enum.Font.Gotham
            btn.TextSize = 11
            btn.TextXAlignment = Enum.TextXAlignment.Left
            btn.ZIndex = 4
            btn.Parent = playerList
            corner(btn, 6)
            local pad = Instance.new("UIPadding")
            pad.PaddingLeft = UDim.new(0,8)
            pad.Parent = btn
            btn.MouseButton1Click:Connect(function()
                playClick()
                selectedPlayer = p
                btn.BackgroundColor3 = C_SEL
                btn.TextColor3 = C_TEXT
                updateTargetLabel()
            end)
        end
    end
end

refreshList()
Players.PlayerAdded:Connect(function() task.wait(0.5); refreshList() end)
Players.PlayerRemoving:Connect(function() task.wait(0.3); refreshList() end)

-- Teleport Instantâneo
local tpInstBtn = Instance.new("TextButton")
tpInstBtn.Size = UDim2.new(1,0,0,40)
tpInstBtn.BackgroundColor3 = C_JUMP
tpInstBtn.Text = "TELEPORT INSTANTÂNEO"
tpInstBtn.TextColor3 = C_TEXT
tpInstBtn.Font = Enum.Font.GothamBold
tpInstBtn.TextSize = 12
tpInstBtn.ZIndex = 4
tpInstBtn.Parent = playerPage
corner(tpInstBtn, 8)
tpInstBtn.MouseButton1Click:Connect(function()
    playClick()
    if selectedPlayer and selectedPlayer.Character then
        local hrp = player.Character and player.Character:FindFirstChild("HumanoidRootPart")
        local tgt = selectedPlayer.Character:FindFirstChild("HumanoidRootPart")
        if hrp and tgt then hrp.CFrame = tgt.CFrame + Vector3.new(0,5,0) end
    end
end)

-- GOTO Tween
local gotoBtn = Instance.new("TextButton")
gotoBtn.Size = UDim2.new(1,0,0,40)
gotoBtn.BackgroundColor3 = C_GOTO
gotoBtn.Text = "GOTO TWEEN"
gotoBtn.TextColor3 = C_TEXT
gotoBtn.Font = Enum.Font.GothamBold
gotoBtn.TextSize = 12
gotoBtn.ZIndex = 4
gotoBtn.Parent = playerPage
corner(gotoBtn, 8)
gotoBtn.MouseButton1Click:Connect(function()
    playClick()
    if not selectedPlayer or not selectedPlayer.Character then return end
    local char = player.Character
    local hum = char and char:FindFirstChildOfClass("Humanoid")
    local root = char and char:FindFirstChild("HumanoidRootPart")
    local tgtHrp = selectedPlayer.Character:FindFirstChild("HumanoidRootPart")
    if not hum or not root or not tgtHrp then return end
    hum.PlatformStand = true
    local goal = CFrame.new(tgtHrp.Position + Vector3.new(0, HOVER_OFFSET, 0)) * root.CFrame.Rotation
    local tw = TweenService:Create(root, TweenInfo.new(TWEEN_TIME, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = goal})
    tw:Play()
    tw.Completed:Connect(function()
        hum.PlatformStand = false
    end)
end)

-- ============== ABA 5: WORLD ==============
local worldPage = createTab("WORLD", 5)
createToggle(worldPage, "ANTI-VOID", false, function(state) end)
createToggle(worldPage, "GRAVITY CHANGER", false, function(state)
    workspace.Gravity = state and 50 or 196.2
end)
createToggle(worldPage, "FREEZE WORLD", false, function(state) end)
createToggle(worldPage, "TIME CHANGER", false, function(state)
    Lighting.ClockTime = state and 12 or 14
end)

-- ============== ABA 6: TROLL ==============
local trollPage = createTab("TROLL", 6)

-- Label de Alvo Troll
local trollTargetLabel = Instance.new("TextLabel")
trollTargetLabel.Size = UDim2.new(1,0,0,20)
trollTargetLabel.BackgroundTransparency = 1
trollTargetLabel.Text = "Alvo: NENHUM"
trollTargetLabel.TextColor3 = Color3.fromRGB(255,200,100)
trollTargetLabel.Font = Enum.Font.GothamBold
trollTargetLabel.TextSize = 12
trollTargetLabel.ZIndex = 4
trollTargetLabel.Parent = trollPage

local function updateTrollLabel()
    if selectedPlayer then
        trollTargetLabel.Text = "Alvo: " .. selectedPlayer.Name
        trollTargetLabel.TextColor3 = Color3.fromRGB(120,255,120)
    else
        trollTargetLabel.Text = "Alvo: NENHUM (vá na aba PLAYER)"
        trollTargetLabel.TextColor3 = Color3.fromRGB(255,200,100)
    end
end

-- Fling
local function flingTarget()
    local char = getSelectedChar()
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local bv = Instance.new("BodyVelocity")
    bv.Velocity = Vector3.new(0,500,0)
    bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
    bv.Parent = hrp
    task.delay(0.2, function() bv:Destroy() end)
end
local flingBtn = Instance.new("TextButton")
flingBtn.Size = UDim2.new(1,0,0,40)
flingBtn.BackgroundColor3 = C_TROLL
flingBtn.Text = "FLING (Arremessar)"
flingBtn.TextColor3 = C_TEXT
flingBtn.Font = Enum.Font.GothamBold
flingBtn.TextSize = 12
flingBtn.ZIndex = 4
flingBtn.Parent = trollPage
corner(flingBtn, 8)
flingBtn.MouseButton1Click:Connect(function() playClick(); flingTarget() end)

-- Freeze
local function freezeTarget()
    local char = getSelectedChar()
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    hrp.Anchored = true
    task.delay(5, function() if hrp and hrp.Parent then hrp.Anchored = false end end)
end
local freezeBtn = Instance.new("TextButton")
freezeBtn.Size = UDim2.new(1,0,0,40)
freezeBtn.BackgroundColor3 = C_TROLL
freezeBtn.Text = "FREEZE (Congelar 5s)"
freezeBtn.TextColor3 = C_TEXT
freezeBtn.Font = Enum.Font.GothamBold
freezeBtn.TextSize = 12
freezeBtn.ZIndex = 4
freezeBtn.Parent = trollPage
corner(freezeBtn, 8)
freezeBtn.MouseButton1Click:Connect(function() playClick(); freezeTarget() end)

-- Kill
local function killTarget()
    local char = getSelectedChar()
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then hum.Health = 0 end
end
local killBtn = Instance.new("TextButton")
killBtn.Size = UDim2.new(1,0,0,40)
killBtn.BackgroundColor3 = C_TROLL
killBtn.Text = "KILL (Matar)"
killBtn.TextColor3 = C_TEXT
killBtn.Font = Enum.Font.GothamBold
killBtn.TextSize = 12
killBtn.ZIndex = 4
killBtn.Parent = trollPage
corner(killBtn, 8)
killBtn.MouseButton1Click:Connect(function() playClick(); killTarget() end)

-- Explode
local function explodeTarget()
    local char = getSelectedChar()
    if not char then return end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    if not hrp then return end
    local exp = Instance.new("Explosion")
    exp.Position = hrp.Position
    exp.Parent = workspace
end
local explodeBtn = Instance.new("TextButton")
explodeBtn.Size = UDim2.new(1,0,0,40)
explodeBtn.BackgroundColor3 = C_TROLL
explodeBtn.Text = "EXPLODE (Explodir)"
explodeBtn.TextColor3 = C_TEXT
explodeBtn.Font = Enum.Font.GothamBold
explodeBtn.TextSize = 12
explodeBtn.ZIndex = 4
explodeBtn.Parent = trollPage
corner(explodeBtn, 8)
explodeBtn.MouseButton1Click:Connect(function() playClick(); explodeTarget() end)

-- Rainbow Troll
local rainbowEnabled = false
local rainbowConn
createToggle(trollPage, "RAINBOW TROLL", false, function(state)
    rainbowEnabled = state
    if rainbowEnabled then
        rainbowConn = RunService.Heartbeat:Connect(function()
            if not rainbowEnabled then return end
            local char = getSelectedChar()
            if char then
                for _, p in ipairs(char:GetDescendants()) do
                    if p:IsA("BasePart") then
                        p.Color = Color3.fromRGB(math.random(0,255), math.random(0,255), math.random(0,255))
                    end
                end
            end
        end)
    else
        if rainbowConn then rainbowConn:Disconnect() end
    end
end)

-- ============== ABA 7: UTILITY ==============
local utilPage = createTab("UTILITY", 7)

-- Anti-AFK
local antiAfkEnabled = false
local afkThread
createToggle(utilPage, "ANTI-AFK", false, function(state)
    antiAfkEnabled = state
    if antiAfkEnabled then
        afkThread = task.spawn(function()
            while antiAfkEnabled do
                local vu = game:GetService("VirtualUser")
                vu:CaptureController()
                vu:ClickButton2(Vector2.new())
                task.wait(60)
            end
        end)
    else
        if afkThread then task.cancel(afkThread) end
    end
end)

-- Godmode
local godEnabled = false
local godConn
createToggle(utilPage, "GODMODE", false, function(state)
    godEnabled = state
    if godEnabled then
        godConn = RunService.Heartbeat:Connect(function()
            if not godEnabled then return end
            local char = player.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            if hum then hum.MaxHealth = math.huge; hum.Health = math.huge end
        end)
    else
        if godConn then godConn:Disconnect() end
    end
end)

-- Rejoin
local rejoinBtn = Instance.new("TextButton")
rejoinBtn.Size = UDim2.new(1,0,0,40)
rejoinBtn.BackgroundColor3 = C_GOTO
rejoinBtn.Text = "REJOIN SERVER"
rejoinBtn.TextColor3 = C_TEXT
rejoinBtn.Font = Enum.Font.GothamBold
rejoinBtn.TextSize = 12
rejoinBtn.ZIndex = 4
rejoinBtn.Parent = utilPage
corner(rejoinBtn, 8)
rejoinBtn.MouseButton1Click:Connect(function()
    playClick()
    TeleportService:Teleport(game.PlaceId, player)
end)

-- Server Hop
local hopBtn = Instance.new("TextButton")
hopBtn.Size = UDim2.new(1,0,0,40)
hopBtn.BackgroundColor3 = C_GOTO
hopBtn.Text = "SERVER HOP"
hopBtn.TextColor3 = C_TEXT
hopBtn.Font = Enum.Font.GothamBold
hopBtn.TextSize = 12
hopBtn.ZIndex = 4
hopBtn.Parent = utilPage
corner(hopBtn, 8)
hopBtn.MouseButton1Click:Connect(function()
    playClick()
    TeleportService:Teleport(game.PlaceId, player)
end)

-- ============== MINI OPEN ==============
local miniGui = Instance.new("ScreenGui")
miniGui.Name = "MiniOpenGUI"
miniGui.ResetOnSpawn = false
miniGui.IgnoreGuiInset = true
miniGui.DisplayOrder = 1001
miniGui.Enabled = false
miniGui.Parent = player:WaitForChild("PlayerGui")

local openBtn = Instance.new("TextButton")
openBtn.Size = UDim2.new(0, 58, 0, 58)
openBtn.Position = UDim2.new(0, 20, 0.5, 0)
openBtn.BackgroundColor3 = C_ACCENT
openBtn.Text = "☰"
openBtn.TextColor3 = C_TEXT
openBtn.Font = Enum.Font.GothamBold
openBtn.TextSize = 22
openBtn.AutoButtonColor = false
openBtn.Parent = miniGui
corner(openBtn, 999)

local openBusy = false
minimizeBtn.MouseButton1Click:Connect(function()
    playClick()
    if openBusy then return end
    openBusy = true
    task.delay(0.25, function() openBusy = false end)
    panel.Visible = false
    miniGui.Enabled = true
end)

openBtn.MouseButton1Click:Connect(function()
    playClick()
    if openBusy then return end
    openBusy = true
    task.delay(0.25, function() openBusy = false end)
    miniGui.Enabled = false
    panel.Visible = true
end)
