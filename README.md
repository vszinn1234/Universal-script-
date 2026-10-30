-- ============================================
-- UNIVERSAL SCRIPT v13.0 (HALLOWEEN + SAVE CONFIG)
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

-- PALETA HALLOWEEN
local C_BG        = Color3.fromRGB(15, 5, 20) -- Roxo escuro
local C_BG2       = Color3.fromRGB(30, 10, 35) -- Roxo médio
local C_ACCENT    = Color3.fromRGB(255, 100, 0) -- Laranja Abóbora
local C_ACCENT2   = Color3.fromRGB(255, 160, 50) -- Laranja claro
local C_TEXT      = Color3.fromRGB(235, 235, 240)
local C_TEXT_DIM  = Color3.fromRGB(180, 150, 190)
local C_ON        = Color3.fromRGB(255, 80, 0)
local C_OFF       = Color3.fromRGB(50, 20, 60)
local C_DANGER    = Color3.fromRGB(200, 30, 30)
local C_SEL       = Color3.fromRGB(180, 50, 200)

local CLICK_SOUND = "rbxassetid://123677837372386"
local CONFIG_FILE = "UniversalHub_Halloween.json"
local savedStates = {}

-- ============== DATA STORE (LOCAL) ==============
local function loadConfig()
    if isfile and isfile(CONFIG_FILE) then
        local suc, res = pcall(function()
            return HttpService:JSONDecode(readfile(CONFIG_FILE))
        end)
        if suc and type(res) == "table" then
            savedStates = res
        end
    end
end

local function saveConfig()
    if writefile then
        local suc, res = pcall(function()
            return HttpService:JSONEncode(savedStates)
        end)
        if suc then
            writefile(CONFIG_FILE, res)
            return true
        end
    end
    return false
end

loadConfig() -- Carrega ao iniciar

-- ============== HELPERS ==============
local function corner(parent, r)
    local c = Instance.new("UICorner")
    c.CornerRadius = UDim.new(0, r or 8)
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
for _, n in ipairs({"UniversalGUI", "MiniOpenGUI"}) do
    local g = player.PlayerGui:FindFirstChild(n)
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
panel.Size = UDim2.new(0, 460, 0, 400)
panel.Position = UDim2.new(0.5, -230, 0.5, -200)
panel.BackgroundColor3 = C_BG
panel.BackgroundTransparency = 0.05
panel.BorderSizePixel = 0
panel.Active = true
panel.Draggable = true
panel.ClipsDescendants = true
panel.ZIndex = 2
panel.Parent = gui
corner(panel, 12)
stroke(panel, C_ACCENT, 1.5, 0.3)

local bgGrad = Instance.new("Frame")
bgGrad.Size = UDim2.new(1, 0, 1, 0)
bgGrad.BackgroundColor3 = C_BG2
bgGrad.BackgroundTransparency = 0.1
bgGrad.ZIndex = 1
bgGrad.Parent = panel
corner(bgGrad, 12)
local grad = Instance.new("UIGradient")
grad.Color = ColorSequence.new(Color3.fromRGB(40, 10, 50), Color3.fromRGB(10, 0, 15))
grad.Rotation = 90
grad.Parent = bgGrad

-- HEADER
local headerBar = Instance.new("Frame")
headerBar.Size = UDim2.new(1, -20, 0, 26)
headerBar.Position = UDim2.new(0, 10, 0, 10)
headerBar.BackgroundTransparency = 1
headerBar.ZIndex = 3
headerBar.Parent = panel

local accentBar = Instance.new("Frame")
accentBar.Size = UDim2.new(0, 3, 0, 16)
accentBar.Position = UDim2.new(0, 0, 0.5, -8)
accentBar.BackgroundColor3 = C_ACCENT
accentBar.BorderSizePixel = 0
accentBar.ZIndex = 3
accentBar.Parent = headerBar
corner(accentBar, 2)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -120, 1, 0)
title.Position = UDim2.new(0, 8, 0, 0)
title.BackgroundTransparency = 1
title.Text = "🎃 HALLOWEEN HUB"
title.TextColor3 = C_ACCENT2
title.Font = Enum.Font.GothamBold
title.TextSize = 13
title.TextXAlignment = Enum.TextXAlignment.Left
title.ZIndex = 3
title.Parent = headerBar

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 28, 0, 22)
minimizeBtn.Position = UDim2.new(1, -28, 0, 2)
minimizeBtn.BackgroundColor3 = C_DANGER
minimizeBtn.Text = "🗕"
minimizeBtn.TextColor3 = C_TEXT
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 12
minimizeBtn.ZIndex = 3
minimizeBtn.Parent = headerBar
corner(minimizeBtn, 6)

-- ============== ABAS (MENOR) ==============
local tabBar = Instance.new("ScrollingFrame")
tabBar.Size = UDim2.new(1, -20, 0, 26)
tabBar.Position = UDim2.new(0, 10, 0, 40)
tabBar.BackgroundTransparency = 1
tabBar.BorderSizePixel = 0
tabBar.ScrollBarThickness = 2
tabBar.ScrollBarImageColor3 = C_ACCENT
tabBar.ScrollingDirection = Enum.ScrollingDirection.X
tabBar.CanvasSize = UDim2.new(0, 1000, 0, 0)
tabBar.ZIndex = 3
tabBar.Parent = panel

local tabLayout = Instance.new("UIListLayout")
tabLayout.FillDirection = Enum.FillDirection.Horizontal
tabLayout.Padding = UDim.new(0, 4)
tabLayout.SortOrder = Enum.SortOrder.LayoutOrder
tabLayout.VerticalAlignment = Enum.VerticalAlignment.Center
tabLayout.Parent = tabBar

local tabContent = Instance.new("Frame")
tabContent.Size = UDim2.new(1, -20, 1, -76)
tabContent.Position = UDim2.new(0, 10, 0, 70)
tabContent.BackgroundTransparency = 1
tabContent.ZIndex = 3
tabContent.Parent = panel

local pages = {}
local tabs = {}

local function createTab(name, order)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 75, 0, 22)
    btn.BackgroundColor3 = C_OFF
    btn.BackgroundTransparency = 0.2
    btn.Text = name
    btn.TextColor3 = C_TEXT_DIM
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 10
    btn.LayoutOrder = order
    btn.AutoButtonColor = false
    btn.ZIndex = 4
    btn.Parent = tabBar
    corner(btn, 5)

    local page = Instance.new("ScrollingFrame")
    page.Size = UDim2.new(1, 0, 1, 0)
    page.BackgroundTransparency = 1
    page.BorderSizePixel = 0
    page.ScrollBarThickness = 2
    page.ScrollBarImageColor3 = C_ACCENT
    page.CanvasSize = UDim2.new(0, 0, 0, 0)
    page.AutomaticCanvasSize = Enum.AutomaticSize.Y
    page.Visible = (order == 1)
    page.ZIndex = 3
    page.Parent = tabContent

    local pageLayout = Instance.new("UIListLayout")
    pageLayout.Padding = UDim.new(0, 4)
    pageLayout.SortOrder = Enum.SortOrder.LayoutOrder
    pageLayout.Parent = page

    tabs[name] = { btn = btn, page = page }
    pages[name] = page

    if order == 1 then
        btn.BackgroundColor3 = C_ACCENT
        btn.TextColor3 = C_BG
    end

    btn.MouseButton1Click:Connect(function()
        playClick()
        for _, t in pairs(tabs) do
            t.page.Visible = false
            t.btn.BackgroundColor3 = C_OFF
            t.btn.TextColor3 = C_TEXT_DIM
        end
        page.Visible = true
        btn.BackgroundColor3 = C_ACCENT
        btn.TextColor3 = C_BG
    end)

    return page
end

-- ============== SISTEMA DE BOTÕES ==============
local function addButton(parent, text, color, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    btn.BackgroundColor3 = color or C_OFF
    btn.Text = text
    btn.TextColor3 = C_TEXT
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    btn.ZIndex = 4
    btn.Parent = parent
    corner(btn, 5)
    btn.MouseButton1Click:Connect(function()
        playClick()
        if callback then callback() end
    end)
    return btn
end

local function addToggle(parent, name, defaultState, callback)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, 0, 0, 30)
    
    -- Checa se tem estado salvo
    if savedStates[name] ~= nil then
        defaultState = savedStates[name]
    else
        savedStates[name] = defaultState
    end

    btn.BackgroundColor3 = defaultState and C_ON or C_OFF
    btn.Text = name .. ": " .. (defaultState and "ON" or "OFF")
    btn.TextColor3 = C_TEXT
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 11
    btn.ZIndex = 4
    btn.Parent = parent
    corner(btn, 5)
    
    local state = defaultState
    -- Callback inicial se estiver ligado
    if state and callback then task.spawn(function() callback(true) end) end

    btn.MouseButton1Click:Connect(function()
        playClick()
        state = not state
        savedStates[name] = state -- Atualiza DataStore local
        btn.Text = name .. ": " .. (state and "ON" or "OFF")
        btn.BackgroundColor3 = state and C_ON or C_OFF
        if callback then callback(state) end
    end)
    return btn
end

-- ============== CATEGORIAS ==============
local CATEGORIES = {
    { name = "COMBAT", order = 1 },
    { name = "VISUAL", order = 2 },
    { name = "MOVEMENT", order = 3 },
    { name = "PLAYER", order = 4 },
    { name = "TROLL", order = 5 },
    { name = "UTILITY", order = 6 },
    { name = "CONFIG", order = 7 },
}

for _, cat in ipairs(CATEGORIES) do
    createTab(cat.name, cat.order)
end

-- ============== ABA 1: COMBAT ==============
local combatPage = pages["COMBAT"]

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
                if target then camera.CFrame = CFrame.new(camera.CFrame.Position, target.Position) end
            end
        end)
    else
        if conn then conn:Disconnect() end
    end
end)

addToggle(combatPage, "KILL AURA", false, function(state)
    local conn
    if state then
        conn = RunService.Heartbeat:Connect(function()
            local _, _, root = getChar()
            if not root then return end
            for _, p in ipairs(Players:GetPlayers()) do
                if p ~= player and p.Character then
                    local hrp = p.Character:FindFirstChild("HumanoidRootPart")
                    if hrp and (root.Position - hrp.Position).Magnitude < 10 then
                        pcall(function()
                            local hitbox = Instance.new("Part")
                            hitbox.Size = Vector3.new(5,5,5); hitbox.Transparency = 1; hitbox.CanCollide = false
                            hitbox.Parent = workspace; hitbox.CFrame = hrp.CFrame
                            firetouchinterest(root, hitbox, 0); task.wait(0.05); firetouchinterest(root, hitbox, 1)
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
        _G.hitboxParts = parts
    else
        if _G.hitboxParts then
            for p, part in pairs(_G.hitboxParts) do part:Destroy() end
        end
    end
end)

-- ============== ABA 2: VISUAL ==============
local visualPage = pages["VISUAL"]

addToggle(visualPage, "PLAYER ESP", false, function(state)
    local highlights = {}
    if state then
        for _, p in ipairs(Players:GetPlayers()) do
            if p ~= player and p.Character then
                local hl = Instance.new("Highlight")
                hl.Adornee = p.Character; hl.FillColor = C_ACCENT
                hl.FillTransparency = 0.5; hl.Parent = p.Character
                highlights[p] = hl
            end
        end
        _G.espHighlights = highlights
    else
        if _G.espHighlights then
            for p, hl in pairs(_G.espHighlights) do hl:Destroy() end
        end
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

addToggle(visualPage, "X-RAY", false, function(state)
    for _, v in ipairs(workspace:GetDescendants()) do
        if v:IsA("BasePart") and v.Name ~= "HumanoidRootPart" then
            v.LocalTransparencyModifier = state and 0.9 or 0
        end
    end
end)

-- ============== ABA 3: MOVEMENT ==============
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
            _G.flyBV, _G.flyBG, _G.flyConn = bv, bg, conn
        end
    else
        if _G.flyConn then _G.flyConn:Disconnect() end
        if _G.flyBV then _G.flyBV:Destroy() end
        if _G.flyBG then _G.flyBG:Destroy() end
    end
end)

addToggle(movePage, "SPEED 120", false, function(state)
    local conn
    if state then
        conn = RunService.Stepped:Connect(function()
            local _, hum = getChar()
            if hum then hum.WalkSpeed = 120 end
        end)
        _G.speedConn = conn
    else
        if _G.speedConn then _G.speedConn:Disconnect() end
        local _, hum = getChar()
        if hum then hum.WalkSpeed = 16 end
    end
end)

addToggle(movePage, "INFINITE JUMP", false, function(state)
    local conn
    if state then
        conn = UIS.JumpRequest:Connect(function()
            local _, hum = getChar()
            if hum then hum:ChangeState(Enum.HumanoidStateType.Jumping) end
        end)
        _G.infJumpConn = conn
    else
        if _G.infJumpConn then _G.infJumpConn:Disconnect() end
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
        _G.noclipConn = conn
    else
        if _G.noclipConn then _G.noclipConn:Disconnect() end
    end
end)

-- ============== ABA 4: PLAYER ==============
local playerPage = pages["PLAYER"]
local selectedPlayer = nil
local targetLabel = Instance.new("TextLabel")
targetLabel.Size = UDim2.new(1,0,0,20); targetLabel.BackgroundTransparency = 1
targetLabel.Text = "Alvo: NENHUM"; targetLabel.TextColor3 = C_ACCENT
targetLabel.Font = Enum.Font.GothamBold; targetLabel.TextSize = 11
targetLabel.Parent = playerPage

local playerList = Instance.new("ScrollingFrame")
playerList.Size = UDim2.new(1,0,0,100); playerList.BackgroundColor3 = C_OFF
playerList.BackgroundTransparency = 0.4; playerList.BorderSizePixel = 0
playerList.ScrollBarThickness = 2; playerList.ScrollBarImageColor3 = C_ACCENT
playerList.CanvasSize = UDim2.new(0,0,0,0); playerList.AutomaticCanvasSize = Enum.AutomaticSize.Y
playerList.Parent = playerPage
corner(playerList, 6)

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0,4); listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = playerList

local function refreshList()
    for _, c in ipairs(playerList:GetChildren()) do
        if c:IsA("TextButton") then c:Destroy() end
    end
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player then
            local btn = Instance.new("TextButton")
            btn.Size = UDim2.new(1,0,0,24); btn.BackgroundColor3 = C_BG2
            btn.Text = p.Name; btn.TextColor3 = C_TEXT_DIM
            btn.Font = Enum.Font.Gotham; btn.TextSize = 10
            btn.TextXAlignment = Enum.TextXAlignment.Left; btn.Parent = playerList
            corner(btn, 4)
            local pad = Instance.new("UIPadding"); pad.PaddingLeft = UDim.new(0,8); pad.Parent = btn
            btn.MouseButton1Click:Connect(function()
                playClick()
                selectedPlayer = p
                btn.BackgroundColor3 = C_SEL; btn.TextColor3 = C_TEXT
                targetLabel.Text = "Alvo: " .. p.Name
                targetLabel.TextColor3 = C_ON
            end)
        end
    end
end

refreshList()
Players.PlayerAdded:Connect(function() task.wait(0.5); refreshList() end)
Players.PlayerRemoving:Connect(function() task.wait(0.3); refreshList() end)

addButton(playerPage, "TELEPORT (Instantâneo)", C_ACCENT, function()
    local _, _, root = getChar()
    local _, _, tgt = getTargetChar()
    if root and tgt then root.CFrame = tgt.CFrame + Vector3.new(0,5,0) end
end)

-- ============== ABA 5: TROLL ==============
local trollPage = pages["TROLL"]

addButton(trollPage, "FLING (Arremessar)", C_ON, function()
    local char = getTargetChar()
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            local bv = Instance.new("BodyVelocity")
            bv.Velocity = Vector3.new(0,500,0); bv.MaxForce = Vector3.new(math.huge, math.huge, math.huge)
            bv.Parent = hrp; task.delay(0.2, function() bv:Destroy() end)
        end
    end
end)

addButton(trollPage, "FREEZE (Congelar)", C_ON, function()
    local char = getTargetChar()
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then
            hrp.Anchored = true
            task.delay(5, function() if hrp and hrp.Parent then hrp.Anchored = false end end)
        end
    end
end)

addButton(trollPage, "KILL (Matar)", C_ON, function()
    local char = getTargetChar()
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.Health = 0 end
    end
end)

addButton(trollPage, "TELEPORT TO VOID", C_DANGER, function()
    local char = getTargetChar()
    if char then
        local hrp = char:FindFirstChild("HumanoidRootPart")
        if hrp then hrp.CFrame = CFrame.new(0, -500, 0) end
    end
end)

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
        _G.rainbowConn = conn
    else
        if _G.rainbowConn then _G.rainbowConn:Disconnect() end
    end
end)

-- ============== ABA 6: UTILITY ==============
local utilPage = pages["UTILITY"]

addToggle(utilPage, "ANTI-AFK", false, function(state)
    if state then
        task.spawn(function()
            while savedStates["ANTI-AFK"] do
                local vu = game:GetService("VirtualUser")
                vu:CaptureController(); vu:ClickButton2(Vector2.new())
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
        _G.godConn = conn
    else
        if _G.godConn then _G.godConn:Disconnect() end
        local _, hum = getChar()
        if hum then hum.MaxHealth = 100; hum.Health = 100 end
    end
end)

addButton(utilPage, "REJOIN SERVER", C_ACCENT, function()
    TeleportService:Teleport(game.PlaceId, player)
end)

-- ============== ABA 7: CONFIG (DATA STORE) ==============
local configPage = pages["CONFIG"]

addButton(configPage, "SALVAR CONFIGURAÇÕES", C_ACCENT, function()
    local success = saveConfig()
    if success then
        targetLabel.Text = "Configurações Salvas!"
        targetLabel.TextColor3 = C_ON
        task.delay(2, function() targetLabel.Text = "Alvo: NENHUM"; targetLabel.TextColor3 = C_ACCENT end)
    end
end)

addButton(configPage, "CARREGAR CONFIGURAÇÕES", C_ACCENT, function()
    loadConfig()
    -- Força o recarregamento da GUI
    gui:Destroy()
    task.wait(0.1)
    -- O loadstring precisaria ser chamado de novo pelo usuário, ou o script pode recriar a GUI (mais complexo).
    -- Simplificando: pede pro cara executar de novo ou apenas avisa.
end)

addButton(configPage, "APAGAR CONFIGURAÇÕES", C_DANGER, function()
    if delfile then
        delfile(CONFIG_FILE)
        savedStates = {}
        targetLabel.Text = "Configurações Apagadas!"
        targetLabel.TextColor3 = C_DANGER
    end
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
openBtn.Size = UDim2.new(0, 50, 0, 50)
openBtn.Position = UDim2.new(0, 15, 0.5, 0)
openBtn.BackgroundColor3 = C_ACCENT
openBtn.Text = "🎃"
openBtn.TextColor3 = C_BG
openBtn.Font = Enum.Font.GothamBold
openBtn.TextSize = 24
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
