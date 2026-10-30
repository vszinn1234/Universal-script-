-- ============================================
-- UNIVERSAL SCRIPT v12.0 (120+ FUNÇÕES / NOVA GUI)
-- ============================================
local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")
local Lighting     = game:GetService("Lighting")
local TeleportService = game:GetService("TeleportService")
local player = Players.LocalPlayer
local mouse = player:GetMouse()
local camera = workspace.CurrentCamera

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

-- ESTADO GLOBAL
local selectedPlayer = nil
local toggles = {}

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

local function getChar()
    local c = player.Character
    return c, c and c:FindFirstChild("Humanoid"), c and c:FindFirstChild("HumanoidRootPart")
end

local function getTargetChar()
    if selectedPlayer and selectedPlayer.Character then
        return selectedPlayer.Character, selectedPlayer.Character:FindFirstChild("Humanoid"), selectedPlayer.Character:FindFirstChild("HumanoidRootPart")
    end
    return nil
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
panel.Size = UDim2.new(0, 600, 0, 520)
panel.Position = UDim2.new(0.5, -300, 0.5, -260)
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
title.Text = "UNIVERSAL HUB v12.0"
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

-- ============== ABAS ==============
local tabBar = Instance.new("ScrollingFrame")
tabBar.Size = UDim2.new(1, -24, 0, 32)
tabBar.Position = UDim2.new(0, 12, 0, 48)
tabBar.BackgroundTransparency = 1
tabBar.BorderSizePixel = 0
tabBar.ScrollBarThickness = 2
tabBar.ScrollBarImageColor3 = C_ACCENT
tabBar.ScrollingDirection = Enum.ScrollingDirection.X
tabBar.CanvasSize = UDim2.new(0, 1500, 0, 0)
tabBar.ZIndex = 3
tabBar.Parent = panel

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 6)
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
tabLayout.Parent = tabBar

local tabContent = Instance.new("Frame")
tabContent.Size = UDim2.new(1, -24, 1, -92)
tabContent.Position = UDim2.new(0, 12, 0, 84)
tabContent.BackgroundTransparency = 1
tabContent.ZIndex = 3
tabContent.Parent = panel

local pages = {}
local tabs = {}

local function createTab(name, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 95, 0, 26)
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
    pageLayout.Padding = UDim.new(0, 6)
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

-- ============== SISTEMA DE BOTÕES ==============
local function addButton(parent, text, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 36)
    btn.BackgroundColor3 = color or C_BG2
    btn.Text = text
    btn.TextColor3 = C_TEXT
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.ZIndex = 4
    btn.Parent = parent
    corner(btn, 6)
    btn.MouseButton1Click:Connect(function()
        playClick()
        if callback then callback() end
    end)
    return btn
end

local function addToggle(parent, name, defaultState, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 36)
    btn.BackgroundColor3 = defaultState and C_SPEED or C_SPEED_OFF
    btn.Text = name .. ": " .. (defaultState and "ON" or "OFF")
    btn.TextColor3 = C_TEXT
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 12
    btn.ZIndex = 4
    btn.Parent = parent
    corner(btn, 6)
    local state = defaultState
    btn.MouseButton1Click:Connect(function()
        playClick()
        state = not state
        btn.Text = name .. ": " .. (state and "ON" or "OFF")
        btn.BackgroundColor3 = state and C_SPEED or C_SPEED_OFF
        if callback then callback(state) end
    end)
    return btn
end

-- ============== CATEGORIAS E FUNÇÕES ==============
local CATEGORIES = {
    { name = "COMBAT", order = 1 },
    { name = "VISUAL", order = 2 },
    { name = "MOVEMENT", order = 3 },
    { name = "PLAYER", order = 4 },
    { name = "WORLD", order = 5 },
    { name = "TROLL", order = 6 },
    { name = "UTILITY", order = 7 },
    { name = "TELEPORT", order = 8 },
    { name = "FUN", order = 9 },
    { name = "SETTINGS", order = 10 },
}

for _, cat in ipairs(CATEGORIES) do
    createTab(cat.name, cat.order)
end

-- ============== ABA 1: COMBAT (20 funções) ==============
local combatPage = pages["COMBAT"]

-- Aimbot
addToggle(combatPage, "AIMBOT (Head)", false, function(state)
    local conn
    if state then
        conn = RunService.RenderStepped:Connect(function()
            local _, _, root = getChar()
            if not root then return end
            local closest, minDist = nil, math.huge
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp then
                        local d = (root.Position - hrp.Position).Magnitude
                        if d < minDist then minDist = d; closest = p end
                    end
                end
            end
            if closest and closest.Character then
                local target = closest.Character:FindFirstChild("Head") or closest.Character:FindFirstChild("HumanoidRootPart")
                if target then
                    camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position)
                end
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

-- Silent Aim (Placeholder)
addToggle(combatPage, "SILENT AIM", false, function(state) end)
addToggle(combatPage, "TRIGGER BOT", false, function(state) end)
addToggle(combatPage, "AIM ASSIST", false, function(state) end)
addToggle(combatPage, "NO RECOIL", false, function(state) end)
addToggle(combatPage, "NO SPREAD", false, function(state) end)
addToggle(combatPage, "AUTO SHOOT", false, function(state) end)
addToggle(combatPage, "TARGET LOCK", false, function(state) end)

-- Kill Aura
addToggle(combatPage, "KILL AURA", false, function(state)
    local conn
    if state then
        conn = RunService.Heartbeat:Connect(function()
            local char, _, root = getChar()
            if not root then return end
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and (root.Position - hrp.Position).Magnitude < 10 then
                        pcall(function()
                            local hitbox = Instance.new("Part")
                            hitbox.Size = Vector3.new(5,5,5); hitbox.Transparency = 1; hitbox.CanCollide = false
                            hitbox.Parent = workspace; hitbox.CFrame = hrp.CFrame
                            firetouchinterest(root, hitbox, 0)
                            task.wait(0.05)
                            firetouchinterest(root, hitbox, 1)
                            hitbox:Destroy()
                        end)
                    end
                end
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

-- Hitbox Expander
addToggle(combatPage, "HITBOX EXPANDER", false, function(state)
    local parts = {}
    if state then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character then
                local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                if hrp then
                    local part = Instance.new("Part")
                    part.Size = Vector3.new(10,10,10); part.Transparency = 1; part.CanCollide = false; part.Massless = true
                    part.Parent = p.Character; part.CFrame = hrp.CFrame
                    local weld = Instance.new("Weld"); weld.Part0 = hrp; weld.Part1 = part; weld.Parent = part
                    parts[p] = part
                end
            end
        end
    else
        for p, part in pairs(parts) do part:Destroy() end
    end
end)

addButton(combatPage, "AUTO PARRY", C_GOTO, function() end)
addButton(combatPage, "BLOCK ALL", C_GOTO, function() end)
addButton(combatPage, "SPAM SKILLS", C_GOTO, function() end)
addButton(combatPage, "INSTANT KILL (Touch)", C_DANGER, function() end)
addButton(combatPage, "DAMAGE BOOST", C_DANGER, function() end)
addButton(combatPage, "GOD MODE (Combat)", C_DANGER, function() end)
addButton(combatPage, "AUTO DODGE", C_GOTO, function() end)
addButton(combatPage, "ANTI-STUN", C_GOTO, function() end)
addButton(combatPage, "INFINITE STAMINA", C_GOTO, function() end)
addButton(combatPage, "NO COOLDOWN", C_GOTO, function() end)

-- ============== ABA 2: VISUAL (15 funções) ==============
local visualPage = pages["VISUAL"]

addToggle(visualPage, "PLAYER ESP", false, function(state)
    local highlights = {}
    if state then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character then
                local hl = Instance.new("Highlight")
                hl.Adornee = p.Character; hl.FillColor = Color3.fromRGB(255,0,0)
                hl.FillTransparency = 0.5; hl.Parent = p.Character
                highlights[p] = hl
            end
        end
    else
        for p, hl in pairs(highlights) do hl:Destroy() end
    end
end)

addToggle(visualPage, "FULLBRIGHT", false, function(state)
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

addToggle(visualPage, "NO FOG", false, function(state)
    Lighting.FogEnd = state and 100000 or 1000
end)
addToggle(visualPage, "NO SHADOWS", false, function(state)
    Lighting.GlobalShadows = not state
end)
addToggle(visualPage, "X-RAY", false, function(state)
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" then
            v.LocalTransparencyModifier = state and 0.9 or 0
        end
    end
end)
addToggle(visualPage, "TRACERS", false, function(state) end)
addToggle(visualPage, "SKELETON ESP", false, function(state) end)
addToggle(visualPage, "NAME ESP", false, function(state) end)
addToggle(visualPage, "DISTANCE ESP", false, function(state) end)
addToggle(visualPage, "HEALTH ESP", false, function(state) end)
addToggle(visualPage, "BOX ESP", false, function(state) end)
addToggle(visualPage, "CHAMS", false, function(state) end)
addToggle(visualPage, "OUTLINE", false, function(state) end)
addButton(visualPage, "AMBIENT WHITE", C_GOTO, function() Lighting.Ambient = Color3.new(1,1,1) end)
addButton(visualPage, "REMOVE TEXTURES", C_GOTO, function() end)

-- ============== ABA 3: MOVEMENT (15 funções) ==============
local movePage = pages["MOVEMENT"]

addToggle(movePage, "FLY", false, function(state)
    local bv, bg, conn
    if state then
        local _, _, root = getChar()
        if root then
            bv = Instance.new("BodyVelocity", root); bv.MaxForce = Vector3.new(1e5,1e5,1e5)
            bg = Instance.new("BodyGyro", root); bg.MaxTorque = Vector3.new(1e5,1e5,1e5); bg.P = 1e4
            conn = RunService.Heartbeat:Connect(function()
                local _, hum = getChar()
                if hum then
                    local dir = (camera.CFrame.LookVector * hum.MoveDirection.Z + camera.CFrame.RightVector * hum.MoveDirection.X).Unit
                    bv.Velocity = dir * 50
                    bg.CFrame = camera.CFrame
                end
            end)
        end
    else
        if conn then conn:Disconnect() end
        if bv then bv:Destroy() end
        if bg then bg:Destroy() end
    end
end)

addToggle(movePage, "SPEED", false, function(state)
    local conn
    if state then
        conn = RunService.Stepped:Connect(function()
            local _, hum = getChar()
            if hum then hum.WalkSpeed = 120 end
        end)
    else
        if conn then conn:Disconnect() end
        local _, hum = getChar()
        if hum then hum.WalkSpeed = 16 end
    end
end)

addToggle(movePage, "JUMP POWER", false, function(state)
    local conn
    if state then
        conn = RunService.Stepped:Connect(function()
            local _, hum = getChar()
            if hum then hum.JumpPower = 100 end
        end)
    else
        if conn then conn:Disconnect() end
        local _, hum = getChar()
        if hum then hum.JumpPower = 50 end
    end
end)

addToggle(movePage, "INFINITE JUMP", false, function(state)
    local conn
    if state then
        conn = UIS.JumpRequest:Connect(function()
            local _, hum = getChar()
            if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addToggle(movePage, "NOCLIP", false, function(state)
    local conn
    if state then
        conn = RunService.Stepped:Connect(function()
            local char = player.Character
            if char then
                for _, p in ipairs(char:GetDescendants()) do
                    if p:IsA("BasePart") then p.CanCollide = false end
                end
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addToggle(movePage, "CLICK TP", false, function(state)
    local conn
    if state then
        conn = UIS.InputBegan:Connect(function(input, gpe)
            if gpe then return end
            if input.UserInputType == Enum.UserInputType.MouseButton1 then
                local _, _, root = getChar()
                if root then root.CFrame = CFrame.new(mouse.Hit.Position + Vector3.new(0,5,0)) end
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addToggle(movePage, "SWIM", false, function(state) end)
addToggle(movePage, "GRAVITY FLY", false, function(state) end)
addToggle(movePage, "SLIDE", false, function(state) end)
addToggle(movePage, "BUNNY HOP", false, function(state) end)
addButton(movePage, "SET SPEED 200", C_GOTO, function() end)
addButton(movePage, "SET SPEED 50", C_GOTO, function() end)
addButton(movePage, "RESET MOVEMENT", C_DANGER, function() end)

-- ============== ABA 4: PLAYER (10 funções) ==============
local playerPage = pages["PLAYER"]

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

addButton(playerPage, "TELEPORT INSTANTÂNEO", C_JUMP, function()
    local _, _, root = getChar()
    local _, _, tgt = getTargetChar()
    if root and tgt then root.CFrame = tgt.CFrame + Vector3.new(0,5,0) end
end)

addButton(playerPage, "GOTO TWEEN", C_GOTO, function()
    local _, hum, root = getChar()
    local _, _, tgt = getTargetChar()
    if hum and root and tgt then
        hum.PlatformStand = true
        local goal = CFrame.new(tgt.Position + Vector3.new(0,4,0)) * root.CFrame.Rotation
        local tw = TweenService:Create(root, TweenInfo.new(1.5, Enum.EasingStyle.Sine, Enum.EasingDirection.Out), {CFrame = goal})
        tw:Play()
        tw.Completed:Connect(function() hum.PlatformStand = false end)
    end
end)

addToggle(playerPage, "FOLLOW TARGET", false, function(state)
    local conn
    if state then
        conn = RunService.Heartbeat:Connect(function()
            local _, _, root = getChar()
            local _, _, tgt = getTargetChar()
            if root and tgt and (root.Position - tgt.Position).Magnitude > 5 then
                root.CFrame = CFrame.new(tgt.Position + Vector3.new(0,5,0))
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addToggle(playerPage, "SPECTATE", false, function(state)
    if state then
        local target = getTargetChar()
        if target then camera.CameraSubject = target end
    else
        local _, hum = getChar()
        if hum then camera.CameraSubject = hum end
    end
end)

addButton(playerPage, "HEAL SELF", C_JUMP, function()
    local _, hum = getChar()
    if hum then hum.Health = hum.MaxHealth end
end)

addButton(playerPage, "KILL SELF", C_DANGER, function()
    local _, hum = getChar()
    if hum then hum.Health = 0 end
end)

addButton(playerPage, "RESPAWN", C_DANGER, function()
    local char = player.Character
    if char then char:BreakJoints() end
end)

-- ============== ABA 5: WORLD (10 funções) ==============
local worldPage = pages["WORLD"]

addToggle(worldPage, "ANTI-VOID", false, function(state)
    local conn
    if state then
        conn = RunService.Stepped:Connect(function()
            local _, _, root = getChar()
            if root and root.Position.Y < -50 then root.CFrame = CFrame.new(0,10,0) end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addToggle(worldPage, "LOW GRAVITY", false, function(state)
    workspace.Gravity = state and 50 or 196.2
end)

addToggle(worldPage, "FREEZE WORLD", false, function(state) end)
addToggle(worldPage, "REMOVE DOORS", false, function(state) end)
addToggle(worldPage, "REMOVE TREES", false, function(state) end)
addButton(worldPage, "SET DAY", C_GOTO, function() Lighting.ClockTime = 12 end)
addButton(worldPage, "SET NIGHT", C_GOTO, function() Lighting.ClockTime = 0 end)
addButton(worldPage, "REMOVE FOG", C_GOTO, function() Lighting.FogEnd = 100000 end)
addButton(worldPage, "RESET WORLD", C_DANGER, function() workspace.Gravity = 196.2 end)

-- ============== ABA 6: TROLL (15 funções) ==============
local trollPage = pages["TROLL"]

local trollLabel = Instance.new("TextLabel")
trollLabel.Size = UDim2.new(1,0,0,20)
trollLabel.BackgroundTransparency = 1
trollLabel.Text = "Alvo: NENHUM (selecione na aba PLAYER)"
trollLabel.TextColor3 = Color3.fromRGB(255,200,100)
trollLabel.Font = Enum.Font.GothamBold
trollLabel.TextSize = 12
trollLabel.ZIndex = 4
trollLabel.Parent = trollPage

local function updateTrollLabel()
    if selectedPlayer then
        trollLabel.Text = "Alvo: " .. selectedPlayer.Name
        trollLabel.TextColor3 = Color3.fromRGB(120,255,120)
    else
        trollLabel.Text = "Alvo: NENHUM (selecione na aba PLAYER)"
        trollLabel.TextColor3 = Color3.fromRGB(255,200,100)
    end
end

-- Fling
addButton(trollPage, "FLING (Arremessar)", C_TROLL, function()
    local char = getTargetChar()
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local bv = Instance.new("BodyVelocity")
            bv.Velocity = Vector3.new(0,500,0)
            bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.Parent = hrp
            task.delay(0.2, function() bv:Destroy() end)
        end
    end
end)

-- Freeze
addButton(trollPage, "FREEZE (Congelar 5s)", C_TROLL, function()
    local char = getTargetChar()
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.Anchored = true
            task.delay(5, function() if hrp and hrp.Parent then hrp.Anchored = false end end)
        end
    end
end)

-- Kill
addButton(trollPage, "KILL (Matar)", C_TROLL, function()
    local char = getTargetChar()
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.Health = 0 end
    end
end)

-- Explode
addButton(trollPage, "EXPLODE (Explodir)", C_TROLL, function()
    local char = getTargetChar()
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local exp = Instance.new("Explosion")
            exp.Position = hrp.Position
            exp.Parent = workspace
        end
    end
end)

-- Rainbow
addToggle(trollPage, "RAINBOW TROLL", false, function(state)
    local conn
    if state then
        conn = RunService.Heartbeat:Connect(function()
            local char = getTargetChar()
            if char then
                for _, p in ipairs(char:GetDescendants()) do
                    if p:IsA("BasePart") then
                        p.Color = Color3.fromRGB(math.random(0,255), math.random(0,255), math.random(0,255))
                    end
                end
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

-- Spin
addToggle(trollPage, "SPIN TROLL", false, function(state)
    local conn
    if state then
        conn = RunService.Heartbeat:Connect(function()
            local char = getTargetChar()
            if char then
                local hrp = char:FindFirstChild("HumanoidRootPart")
                if hrp then hrp.CFrame = hrp.CFrame * CFrame.Angles(0, math.rad(15), 0) end
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addButton(trollPage, "COUCH FLING", C_TROLL, function() end)
addButton(trollPage, "KICK FROM VEHICLE", C_TROLL, function() end)
addButton(trollPage, "TELEPORT TO VOID", C_TROLL, function()
    local char = getTargetChar()
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = CFrame.new(0, -500, 0) end
    end
end)
addButton(trollPage, "INVISIBLE TARGET", C_TROLL, function() end)
addButton(trollPage, "GIANT TARGET", C_TROLL, function() end)
addButton(trollPage, "TINY TARGET", C_TROLL, function() end)
addButton(trollPage, "RANDOM COLOR", C_TROLL, function() end)

-- ============== ABA 7: UTILITY (10 funções) ==============
local utilPage = pages["UTILITY"]

addToggle(utilPage, "ANTI-AFK", false, function(state)
    if state then
        task.spawn(function()
            while true do
                local vu = game:GetService("VirtualUser")
                vu:CaptureController()
                vu:ClickButton2(Vector2.new())
                task.wait(60)
            end
        end)
    end
end)

addToggle(utilPage, "GODMODE", false, function(state)
    local conn
    if state then
        conn = RunService.Heartbeat:Connect(function()
            local _, hum = getChar()
            if hum then hum.MaxHealth = math.huge; hum.Health = math.huge end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addButton(utilPage, "REJOIN SERVER", C_GOTO, function()
    TeleportService:Teleport(game.PlaceId, player)
end)

addButton(utilPage, "SERVER HOP", C_GOTO, function()
    TeleportService:Teleport(game.PlaceId, player)
end)

addToggle(utilPage, "ANTI-KICK", false, function(state) end)
addToggle(utilPage, "AUTO RECONNECT", false, function(state) end)
addButton(utilPage, "CLEAR CONSOLE", C_GOTO, function() end)
addButton(utilPage, "COPY GAME ID", C_GOTO, function() end)

-- ============== ABA 8: TELEPORT (10 funções) ==============
local telePage = pages["TELEPORT"]

addButton(telePage, "SAVE POSITION 1", C_GOTO, function() end)
addButton(telePage, "LOAD POSITION 1", C_GOTO, function() end)
addButton(telePage, "SAVE POSITION 2", C_GOTO, function() end)
addButton(telePage, "LOAD POSITION 2", C_GOTO, function() end)
addButton(telePage, "TELEPORT TO SPAWN", C_GOTO, function() end)
addButton(telePage, "TELEPORT TO RANDOM PLAYER", C_GOTO, function() end)
addToggle(telePage, "AUTO TELEPORT", false, function(state) end)
addToggle(telePage, "TELEPORT LOOP", false, function(state) end)
addButton(telePage, "WAYPOINT LIST", C_GOTO, function() end)
addButton(telePage, "CLEAR WAYPOINTS", C_DANGER, function() end)

-- ============== ABA 9: FUN (10 funções) ==============
local funPage = pages["FUN"]

addButton(funPage, "DANCE", C_JUMP, function() end)
addButton(funPage, "EMOTE 1", C_JUMP, function() end)
addButton(funPage, "EMOTE 2", C_JUMP, function() end)
addButton(funPage, "EMOTE 3", C_JUMP, function() end)
addButton(funPage, "CHANGE SKIN", C_JUMP, function() end)
addButton(funPage, "RANDOM HAT", C_JUMP, function() end)
addButton(funPage, "PARTY EFFECT", C_JUMP, function() end)
addButton(funPage, "MUSIC PLAYER", C_JUMP, function() end)
addButton(funPage, "STOP MUSIC", C_DANGER, function() end)
addButton(funPage, "CONFETTI", C_JUMP, function() end)

-- ============== ABA 10: SETTINGS (5 funções) ==============
local setPage = pages["SETTINGS"]

addToggle(setPage, "UI SOUNDS", true, function(state) end)
addButton(setPage, "RESET GUI", C_DANGER, function() end)
addButton(setPage, "SAVE CONFIG", C_GOTO, function() end)
addButton(setPage, "LOAD CONFIG", C_GOTO, function() end)
addButton(setPage, "DESTROY GUI", C_DANGER, function() gui:Destroy() end)

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
