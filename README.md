-- ============================================
-- GOTO + JUMP + SPEED · UI Polida + Animada
-- COMPATÍVEL: Delta/Codex/Fluxus/Arceus/Wave/KRNL
-- ============================================
local Players      = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService   = game:GetService("RunService")
local UIS          = game:GetService("UserInputService")
local player = Players.LocalPlayer

-- CONFIG
local TWEEN_TIME     = 1.5
local HOVER_OFFSET   = 4
local RANDOM_OFFSET  = 2
local USE_SWIM_STATE = true
local USE_NOCLIP     = true
local JUMP_VELOCITY  = 50
local JUMP_COOLDOWN  = 0.15
local JUMP_HOLD_RATE = 0.25
local SPEED_DEFAULT  = 120
local ANIM_TIME      = 0.25

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

-- ============== NOCLIP ==============
local noclipActive = false
local noclipConn = nil

local function applyNoclip()
    if not noclipActive then return end
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    if hum then
        local ok = pcall(function() hum:ChangeState(11) end)
        if not ok then
            for _, p in ipairs(char:GetDescendants()) do
                if p:IsA("BasePart") then p.CanCollide = false end
            end
        end
    end
end

local function startNoclip()
    if noclipActive then return end
    noclipActive = true
    local char = player.Character
    if char then
        local root = char:FindFirstChild("HumanoidRootPart")
        if root then pcall(function() root:SetNetworkOwner(player) end) end
    end
    noclipConn = RunService.Stepped:Connect(applyNoclip)
end

local function stopNoclip()
    if not noclipActive then return end
    noclipActive = false
    if noclipConn then noclipConn:Disconnect(); noclipConn = nil end
end

-- ============== LIMPEZA ==============
local cleanup = { "GotoPlayerGUI", "FloatJumpGUI", "MiniOpenGUI" }
for i = 1, #cleanup do
    local g = player.PlayerGui:FindFirstChild(cleanup[i])
    if g then g:Destroy() end
end

-- ============== HELPERS VISUAIS ==============
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
    return nil
end

local function gradient(parent, c1, c2, rot)
    local ok, g = pcall(function()
        local gr = Instance.new("UIGradient")
        gr.Color = ColorSequence.new(c1, c2)
        gr.Rotation = rot or 90
        gr.Parent = parent
        return gr
    end)
    if ok then return g end
    return nil
end

-- ============== GUI PRINCIPAL ==============
local gui = Instance.new("ScreenGui")
gui.Name = "GotoPlayerGUI"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true
gui.DisplayOrder = 999
gui.Parent = player:WaitForChild("PlayerGui")

local panel = Instance.new("Frame")
panel.Size = UDim2.new(0, 500, 0, 200)
panel.Position = UDim2.new(0.5, -250, 0, 40)
panel.BackgroundColor3 = C_BG
panel.BackgroundTransparency = 1
panel.BorderSizePixel = 0
panel.Active = true
panel.Draggable = true
panel.ClipsDescendants = true
panel.Parent = gui
corner(panel, 16)
stroke(panel, C_ACCENT, 1.3, 0.4)
gradient(panel, C_BG2, C_BG, 90)

local panelPad = Instance.new("UIPadding")
panelPad.PaddingTop = UDim.new(0, 12)
panelPad.PaddingBottom = UDim.new(0, 12)
panelPad.PaddingLeft = UDim.new(0, 12)
panelPad.PaddingRight = UDim.new(0, 12)
panelPad.Parent = panel

-- HEADER
local headerBar = Instance.new("Frame")
headerBar.Size = UDim2.new(1, -24, 0, 30)
headerBar.Position = UDim2.new(0, 12, 0, 12)
headerBar.BackgroundTransparency = 1
headerBar.ZIndex = 2
headerBar.Parent = panel

local accentBar = Instance.new("Frame")
accentBar.Size = UDim2.new(0, 3, 0, 18)
accentBar.Position = UDim2.new(0, 0, 0.5, -9)
accentBar.BackgroundColor3 = C_ACCENT
accentBar.BorderSizePixel = 0
accentBar.ZIndex = 2
accentBar.Parent = headerBar
corner(accentBar, 2)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -180, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.BackgroundTransparency = 1
title.Text = "CONTROLE"
title.TextColor3 = C_ACCENT2
title.Font = Enum.Font.GothamBold
title.TextSize = 14
title.TextXAlignment = Enum.TextXAlignment.Left
title.ZIndex = 2
title.Parent = headerBar

local refreshBtn = Instance.new("TextButton")
refreshBtn.Size = UDim2.new(0, 34, 0, 24)
refreshBtn.Position = UDim2.new(1, -78, 0, 2)
refreshBtn.BackgroundColor3 = C_GOTO
refreshBtn.Text = "↻"
refreshBtn.TextColor3 = C_TEXT
refreshBtn.Font = Enum.Font.GothamBold
refreshBtn.TextSize = 14
refreshBtn.ZIndex = 2
refreshBtn.Parent = headerBar
corner(refreshBtn, 7)

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 34, 0, 24)
minimizeBtn.Position = UDim2.new(1, -38, 0, 2)
minimizeBtn.BackgroundColor3 = C_DANGER
minimizeBtn.Text = "🗕"
minimizeBtn.TextColor3 = C_TEXT
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.TextSize = 14
minimizeBtn.ZIndex = 2
minimizeBtn.Parent = headerBar
corner(minimizeBtn, 7)

-- LAYOUT INTERNO
local inner = Instance.new("Frame")
inner.Size = UDim2.new(1, -24, 1, -56)
inner.Position = UDim2.new(0, 12, 0, 52)
inner.BackgroundTransparency = 1
inner.Parent = panel

local innerH = Instance.new("UIListLayout")
innerH.FillDirection = Enum.FillDirection.Horizontal
innerH.Padding = UDim.new(0, 12)
innerH.SortOrder = Enum.SortOrder.LayoutOrder
innerH.Parent = inner

-- COLUNA ESQUERDA
local leftCol = Instance.new("Frame")
leftCol.Size = UDim2.new(0, 260, 1, 0)
leftCol.BackgroundColor3 = C_BG2
leftCol.BackgroundTransparency = 0.4
leftCol.BorderSizePixel = 0
leftCol.LayoutOrder = 1
leftCol.Parent = inner
corner(leftCol, 10)

local leftPad = Instance.new("UIPadding")
leftPad.PaddingTop = UDim.new(0, 8)
leftPad.PaddingBottom = UDim.new(0, 8)
leftPad.PaddingLeft = UDim.new(0, 8)
leftPad.PaddingRight = UDim.new(0, 8)
leftPad.Parent = leftCol

local leftV = Instance.new("UIListLayout")
leftV.Padding = UDim.new(0, 6)
leftV.SortOrder = Enum.SortOrder.LayoutOrder
leftV.Parent = leftCol

local status = Instance.new("TextLabel")
status.Size = UDim2.new(1, 0, 0, 16)
status.BackgroundTransparency = 1
status.Text = "pronto"
status.TextColor3 = C_TEXT_DIM
status.Font = Enum.Font.Gotham
status.TextSize = 11
status.TextXAlignment = Enum.TextXAlignment.Left
status.LayoutOrder = 1
status.Parent = leftCol

local listFrame = Instance.new("ScrollingFrame")
listFrame.Size = UDim2.new(1, 0, 0, 130)
listFrame.BackgroundColor3 = C_BG
listFrame.BackgroundTransparency = 0.3
listFrame.BorderSizePixel = 0
listFrame.ScrollBarThickness = 3
listFrame.ScrollBarImageColor3 = C_ACCENT
listFrame.CanvasSize = UDim2.new(0, 0, 0, 0)
listFrame.LayoutOrder = 2
listFrame.Parent = leftCol
corner(listFrame, 8)

local listLayout = Instance.new("UIListLayout")
listLayout.Padding = UDim.new(0, 4)
listLayout.SortOrder = Enum.SortOrder.LayoutOrder
listLayout.Parent = listFrame

local listPad = Instance.new("UIPadding")
listPad.PaddingTop = UDim.new(0, 4)
listPad.PaddingBottom = UDim.new(0, 4)
listPad.PaddingLeft = UDim.new(0, 4)
listPad.PaddingRight = UDim.new(0, 4)
listPad.Parent = listFrame

local selectedInfo = Instance.new("TextLabel")
selectedInfo.Size = UDim2.new(1, 0, 0, 16)
selectedInfo.BackgroundTransparency = 1
selectedInfo.Text = "selecionado: nenhum"
selectedInfo.TextColor3 = Color3.fromRGB(255, 200, 100)
selectedInfo.Font = Enum.Font.Code
selectedInfo.TextSize = 11
selectedInfo.TextXAlignment = Enum.TextXAlignment.Left
selectedInfo.LayoutOrder = 3
selectedInfo.Parent = leftCol

-- COLUNA DIREITA
local rightCol = Instance.new("Frame")
rightCol.Size = UDim2.new(0, 260, 1, 0)
rightCol.BackgroundTransparency = 1
rightCol.LayoutOrder = 2
rightCol.Parent = inner

local rightV = Instance.new("UIListLayout")
rightV.Padding = UDim.new(0, 8)
rightV.SortOrder = Enum.SortOrder.LayoutOrder
rightV.Parent = rightCol

local gotoBtn = Instance.new("TextButton")
gotoBtn.Size = UDim2.new(1, 0, 0, 44)
gotoBtn.BackgroundColor3 = C_GOTO
gotoBtn.Text = "IR ATÉ O JOGADOR"
gotoBtn.TextColor3 = C_TEXT
gotoBtn.Font = Enum.Font.GothamBold
gotoBtn.TextSize = 13
gotoBtn.LayoutOrder = 1
gotoBtn.Parent = rightCol
corner(gotoBtn, 10)
stroke(gotoBtn, C_ACCENT2, 1.2, 0.4)
gradient(gotoBtn, Color3.fromRGB(50, 160, 240), C_GOTO, 90)

local speedBox = Instance.new("Frame")
speedBox.Size = UDim2.new(1, 0, 0, 80)
speedBox.BackgroundColor3 = C_BG2
speedBox.BackgroundTransparency = 0.4
speedBox.BorderSizePixel = 0
speedBox.LayoutOrder = 2
speedBox.Parent = rightCol
corner(speedBox, 10)

local speedPad = Instance.new("UIPadding")
speedPad.PaddingTop = UDim.new(0, 8)
speedPad.PaddingBottom = UDim.new(0, 8)
speedPad.PaddingLeft = UDim.new(0, 8)
speedPad.PaddingRight = UDim.new(0, 8)
speedPad.Parent = speedBox

local speedBtn = Instance.new("TextButton")
speedBtn.Size = UDim2.new(1, 0, 0, 34)
speedBtn.BackgroundColor3 = C_SPEED_OFF
speedBtn.Text = "SPEED: OFF"
speedBtn.TextColor3 = C_TEXT
speedBtn.Font = Enum.Font.GothamBold
speedBtn.TextSize = 12
speedBtn.Parent = speedBox
corner(speedBtn, 8)
stroke(speedBtn, C_SPEED, 1, 0.5)

local speedSlider = Instance.new("TextBox")
speedSlider.Size = UDim2.new(1, -60, 0, 22)
speedSlider.Position = UDim2.new(0, 0, 0, 42)
speedSlider.BackgroundColor3 = C_BG
speedSlider.BackgroundTransparency = 0.2
speedSlider.Text = tostring(SPEED_DEFAULT)
speedSlider.TextColor3 = Color3.fromRGB(220, 200, 255)
speedSlider.Font = Enum.Font.Code
speedSlider.TextSize = 11
speedSlider.ClearTextOnFocus = false
speedSlider.Parent = speedBox
corner(speedSlider, 6)

local speedValue = Instance.new("TextLabel")
speedValue.Size = UDim2.new(0, 50, 0, 22)
speedValue.Position = UDim2.new(1, -50, 0, 42)
speedValue.BackgroundTransparency = 1
speedValue.Text = tostring(SPEED_DEFAULT)
speedValue.TextColor3 = Color3.fromRGB(200, 160, 255)
speedValue.Font = Enum.Font.Code
speedValue.TextSize = 11
speedValue.TextXAlignment = Enum.TextXAlignment.Right
speedValue.Parent = speedBox

local jumpBtn = Instance.new("TextButton")
jumpBtn.Size = UDim2.new(1, 0, 0, 44)
jumpBtn.BackgroundColor3 = C_JUMP
jumpBtn.Text = "⬆  PULAR"
jumpBtn.TextColor3 = C_TEXT
jumpBtn.Font = Enum.Font.GothamBold
jumpBtn.TextSize = 13
jumpBtn.LayoutOrder = 3
jumpBtn.Parent = rightCol
corner(jumpBtn, 10)
stroke(jumpBtn, Color3.fromRGB(150, 255, 180), 1.2, 0.4)
gradient(jumpBtn, Color3.fromRGB(60, 220, 130), C_JUMP, 90)

-- MINI OPEN
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

local dragOpen = false
local dragStartO, startPosO
openBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch
        or input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragOpen = true
        dragStartO = input.Position
        startPosO = openBtn.Position
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragOpen = false
            end
        end)
    end
end)
UIS.InputChanged:Connect(function(input)
    if not dragOpen then return end
    if input.UserInputType == Enum.UserInputType.Touch
        or input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStartO
        openBtn.Position = UDim2.new(
            startPosO.X.Scale, startPosO.X.Offset + delta.X,
            startPosO.Y.Scale, startPosO.Y.Offset + delta.Y
        )
    end
end)

-- ANIMAÇÕES
local function animateShow()
    panel.Visible = true
    panel.BackgroundTransparency = 1
    panel.Size = UDim2.new(0, 500, 0, 200)
    panel.Position = UDim2.new(0.5, -250, 0, 40)
    local ti = TweenInfo.new(ANIM_TIME, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)
    TweenService:Create(panel, ti, {
        Size = UDim2.new(0, 560, 0, 230),
        Position = UDim2.new(0.5, -280, 0, 20),
        BackgroundTransparency = 0
    }):Play()
end

local function animateHide()
    local ti = TweenInfo.new(ANIM_TIME * 0.8, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
    local t = TweenService:Create(panel, ti, {
        Size = UDim2.new(0, 400, 0, 160),
        Position = UDim2.new(0.5, -200, 0, 50),
        BackgroundTransparency = 1
    })
    t:Play()
    t.Completed:Connect(function()
        panel.Visible = false
    end)
end

local function animateOpenBtnShow()
    openBtn.Size = UDim2.new(0, 30, 0, 30)
    openBtn.BackgroundTransparency = 0.6
    local ti = TweenInfo.new(ANIM_TIME, Enum.EasingStyle.Back, Enum.EasingDirection.Out)
    TweenService:Create(openBtn, ti, {
        Size = UDim2.new(0, 58, 0, 58),
        BackgroundTransparency = 0
    }):Play()
end

local function animateOpenBtnHide()
    local ti = TweenInfo.new(ANIM_TIME * 0.6, Enum.EasingStyle.Quart, Enum.EasingDirection.In)
    local t = TweenService:Create(openBtn, ti, {
        Size = UDim2.new(0, 0, 0, 0),
        BackgroundTransparency = 1
    })
    t:Play()
    t.Completed:Connect(function()
        miniGui.Enabled = false
    end)
end

local miniBusy = false
local function onMinimize()
    if miniBusy then return end
    miniBusy = true
    task.delay(ANIM_TIME + 0.1, function() miniBusy = false end)
    animateHide()
    miniGui.Enabled = true
    animateOpenBtnShow()
end

local function onOpen()
    if miniBusy then return end
    miniBusy = true
    task.delay(ANIM_TIME + 0.1, function() miniBusy = false end)
    animateOpenBtnHide()
    animateShow()
end

minimizeBtn.Activated:Connect(onMinimize)
minimizeBtn.MouseButton1Click:Connect(onMinimize)
openBtn.Activated:Connect(onOpen)
openBtn.MouseButton1Click:Connect(onOpen)

-- SELETOR
local selectedPlayer = nil

local function clearList()
    for _, c in ipairs(listFrame:GetChildren()) do
        if c:IsA("TextButton") then c:Destroy() end
    end
end

local function highlightSelected()
    for _, c in ipairs(listFrame:GetChildren()) do
        if c:IsA("TextButton") then
            if selectedPlayer and c.Name == "P_" .. selectedPlayer.Name then
                c.BackgroundColor3 = C_SEL
                c.BackgroundTransparency = 0.1
                c.TextColor3 = C_TEXT
            else
                c.BackgroundColor3 = C_BG2
                c.BackgroundTransparency = 0.5
                c.TextColor3 = C_TEXT_DIM
            end
        end
    end
end

local function refreshList()
    clearList()
    local count = 0
    for _, p in ipairs(Players:GetPlayers()) do
        if p ~= player then
            count = count + 1
            local btn = Instance.new("TextButton")
            btn.Name = "P_" .. p.Name
            btn.Size = UDim2.new(1, 0, 0, 26)
            btn.BackgroundColor3 = C_BG2
            btn.BackgroundTransparency = 0.5
            btn.Text = p.Name
            btn.TextColor3 = C_TEXT_DIM
            btn.Font = Enum.Font.Gotham
            btn.TextSize = 11
            btn.TextXAlignment = Enum.TextXAlignment.Left
            btn.AutoButtonColor = false
            btn.Parent = listFrame
            corner(btn, 6)
            local pad = Instance.new("UIPadding")
            pad.PaddingLeft = UDim.new(0, 8)
            pad.Parent = btn
            local function selectThis()
                selectedPlayer = p
                selectedInfo.Text = "selecionado: " .. p.Name
                selectedInfo.TextColor3 = Color3.fromRGB(120, 255, 120)
                status.Text = "pronto pra ir"
                status.TextColor3 = Color3.fromRGB(120, 255, 120)
                highlightSelected()
            end
            btn.Activated:Connect(selectThis)
            btn.MouseButton1Click:Connect(selectThis)
        end
    end
    if selectedPlayer and not Players:FindFirstChild(selectedPlayer.Name) then
        selectedPlayer = nil
        selectedInfo.Text = "selecionado: nenhum (saiu)"
        selectedInfo.TextColor3 = Color3.fromRGB(255, 150, 100)
    end
    highlightSelected()
    status.Text = "jogadores: " .. count
    status.TextColor3 = C_TEXT_DIM
end

local function findTargetCF()
    if not selectedPlayer or not selectedPlayer.Parent then return nil end
    local char = selectedPlayer.Character
    if not char then return nil end
    local hrp = char:FindFirstChild("HumanoidRootPart")
    return hrp and hrp.CFrame
end

local refreshBusy = false
local function onRefresh()
    if refreshBusy then return end
    refreshBusy = true
    task.delay(0.3, function() refreshBusy = false end)
    refreshList()
end
refreshBtn.Activated:Connect(onRefresh)
refreshBtn.MouseButton1Click:Connect(onRefresh)
Players.PlayerAdded:Connect(function() task.wait(0.5); refreshList() end)
Players.PlayerRemoving:Connect(function(p)
    if selectedPlayer == p then
        selectedPlayer = nil
        selectedInfo.Text = "selecionado: nenhum (saiu)"
        selectedInfo.TextColor3 = Color3.fromRGB(255, 150, 100)
    end
    task.wait(0.3); refreshList()
end)

-- SPEED
local speedActive = false
local baseWalkSpeed = 16
local speedConn = nil
local targetSpeed = SPEED_DEFAULT

local function applySpeedBoost()
    if not speedActive then return end
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end
    hum.WalkSpeed = targetSpeed
    local move = hum.MoveDirection
    if move.Magnitude > 0.1 then
        local desired = move * targetSpeed
        local cur = root.AssemblyLinearVelocity
        root.AssemblyLinearVelocity = Vector3.new(desired.X, cur.Y, desired.Z)
    end
end

local function startSpeed()
    if speedActive then return end
    speedActive = true
    local char = player.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then
            baseWalkSpeed = hum.WalkSpeed
            local ti = TweenInfo.new(0.4, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
            TweenService:Create(hum, ti, { WalkSpeed = targetSpeed }):Play()
        end
    end
    speedConn = RunService.Stepped:Connect(applySpeedBoost)
    speedBtn.Text = "SPEED: ON (" .. targetSpeed .. ")"
    speedBtn.BackgroundColor3 = C_SPEED
end

local function stopSpeed()
    if not speedActive then return end
    speedActive = false
    if speedConn then speedConn:Disconnect(); speedConn = nil end
    local char = player.Character
    if char then
        local hum = char:FindFirstChildOfClass("Humanoid")
        if hum then hum.WalkSpeed = baseWalkSpeed end
    end
    speedBtn.Text = "SPEED: OFF"
    speedBtn.BackgroundColor3 = C_SPEED_OFF
end

local speedBusy = false
local function onSpeedToggle()
    if speedBusy then return end
    speedBusy = true
    task.delay(0.2, function() speedBusy = false end)
    if speedActive then stopSpeed() else startSpeed() end
end
speedBtn.Activated:Connect(onSpeedToggle)
speedBtn.MouseButton1Click:Connect(onSpeedToggle)

speedSlider.FocusLost:Connect(function()
    local n = tonumber(speedSlider.Text)
    if n and n >= 16 and n <= 1000 then
        targetSpeed = math.floor(n)
        speedValue.Text = tostring(targetSpeed)
        if speedActive then
            local char = player.Character
            if char then
                local hum = char:FindFirstChildOfClass("Humanoid")
                if hum then hum.WalkSpeed = targetSpeed end
            end
            speedBtn.Text = "SPEED: ON (" .. targetSpeed .. ")"
        end
    else
        speedSlider.Text = tostring(targetSpeed)
    end
end)

-- JUMP
local lastJumpTime = 0
local function doJump()
    local now = tick()
    if (now - lastJumpTime) < JUMP_COOLDOWN then return end
    lastJumpTime = now
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end
    local wasNoclip = noclipActive
    noclipActive = false
    local savedPS = hum.PlatformStand
    hum.PlatformStand = false
    local jumpVel = JUMP_VELOCITY
    pcall(function()
        if hum.UseJumpPower == false and hum.JumpHeight and hum.JumpHeight > 0 then
            jumpVel = math.sqrt(2 * workspace.Gravity * hum.JumpHeight)
        elseif hum.JumpPower and hum.JumpPower > 0 then
            jumpVel = hum.JumpPower
        end
    end)
    pcall(function() hum:ChangeState(Enum.HumanoidStateType.Jumping) end)
    pcall(function() hum.Jump = true end)
    root.AssemblyLinearVelocity = Vector3.new(
        root.AssemblyLinearVelocity.X, jumpVel, root.AssemblyLinearVelocity.Z
    )
    task.delay(0.15, function()
        if hum and hum.Parent then hum.PlatformStand = savedPS end
        noclipActive = wasNoclip
    end)
end

local holdingJump = false
jumpBtn.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch
        or input.UserInputType == Enum.UserInputType.MouseButton1 then
        holdingJump = true
        task.spawn(function()
            doJump()
            while holdingJump do
                task.wait(JUMP_HOLD_RATE)
                if holdingJump then doJump() end
            end
        end)
    end
end)
jumpBtn.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.Touch
        or input.UserInputType == Enum.UserInputType.MouseButton1 then
        holdingJump = false
    end
end)

-- GOTO
local running = false
local currentTween = nil

local function doGoto()
    if running then return end
    if not selectedPlayer then
        status.Text = "selecione um jogador primeiro"
        status.TextColor3 = Color3.fromRGB(255, 180, 60)
        return
    end
    if not selectedPlayer.Parent then
        status.Text = "jogador saiu do servidor"
        status.TextColor3 = C_DANGER
        selectedPlayer = nil
        selectedInfo.Text = "selecionado: nenhum"
        return
    end
    local char = player.Character
    if not char then return end
    local hum = char:FindFirstChildOfClass("Humanoid")
    local root = char:FindFirstChild("HumanoidRootPart")
    if not hum or not root then return end
    local tgtCF = findTargetCF()
    if not tgtCF then
        status.Text = "alvo sem personagem"
        status.TextColor3 = Color3.fromRGB(255, 180, 60)
        return
    end
    running = true
    status.Text = "calculando rota..."
    status.TextColor3 = C_ACCENT2
    pcall(function() root:SetNetworkOwner(player) end)
    if USE_SWIM_STATE then
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Swimming) end)
        task.wait(0.8)
    end
    hum.PlatformStand = true
    root.AssemblyLinearVelocity  = Vector3.zero
    root.AssemblyAngularVelocity = Vector3.zero
    root.Anchored = false
    local randOff = Vector3.new(
        math.random(-RANDOM_OFFSET, RANDOM_OFFSET),
        HOVER_OFFSET,
        math.random(-RANDOM_OFFSET, RANDOM_OFFSET)
    )
    local goalCF = CFrame.new(tgtCF.Position + randOff) * root.CFrame.Rotation
    local ti = TweenInfo.new(TWEEN_TIME, Enum.EasingStyle.Sine, Enum.EasingDirection.Out)
    currentTween = TweenService:Create(root, ti, { CFrame = goalCF })
    currentTween:Play()
    status.Text = "voando até " .. selectedPlayer.Name .. "..."
    status.TextColor3 = Color3.fromRGB(120, 255, 150)
    local start = tick()
    while running do
        if not root or not root.Parent then break end
        if not selectedPlayer or not selectedPlayer.Parent then break end
        if (root.Position - goalCF.Position).Magnitude < 2 then break end
        if (tick() - start) > (TWEEN_TIME + 1.5) then break end
        task.wait(0.05)
    end
    currentTween = nil
    if root and root.Parent then
        root.AssemblyLinearVelocity  = Vector3.zero
        root.AssemblyAngularVelocity = Vector3.zero
    end
    if hum and hum.Parent then
        hum.PlatformStand = false
        pcall(function() hum:ChangeState(Enum.HumanoidStateType.Running) end)
    end
    status.Text = "chegou!"
    status.TextColor3 = Color3.fromRGB(120, 255, 120)
    running = false
end

local gotoBusy = false
local function onGoto()
    if gotoBusy or running then return end
    gotoBusy = true
    task.delay(0.5, function() gotoBusy = false end)
    if USE_NOCLIP then startNoclip() end
    doGoto()
end
gotoBtn.Activated:Connect(onGoto)
gotoBtn.MouseButton1Click:Connect(onGoto)

-- INIT
refreshList()
animateShow()
