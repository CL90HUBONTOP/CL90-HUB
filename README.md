-- [[ CL90 HUB V1 - FULL BUILD ]] --
local Players, RunService, CoreGui, Lighting, PathfindingService, UserInputService, Workspace = game:GetService("Players"), game:GetService("RunService"), game:GetService("CoreGui"), game:GetService("Lighting"), game:GetService("PathfindingService"), game:GetService("UserInputService"), game:GetService("Workspace")
local LocalPlayer, Camera = Players.LocalPlayer, Workspace.CurrentCamera

local ScreenGui = Instance.new("ScreenGui", CoreGui)
ScreenGui.Name = "CL90HubV1_RedBlue"
ScreenGui.ResetOnSpawn = false

local function makeDraggable(frame, handle, onDragEnd)
    handle = handle or frame
    local dragging, dragInput, dragStart, startPos
    handle.InputBegan:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch) and not frame:GetAttribute("DragLocked") then
            dragging, dragStart, startPos = true, input.Position, frame.Position
            input.Changed:Connect(function()
                if input.UserInputState == Enum.UserInputState.End and dragging then
                    dragging = false
                    if onDragEnd then onDragEnd() end
                end
            end)
        end
    end)
    handle.InputChanged:Connect(function(input)
        if (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) and not frame:GetAttribute("DragLocked") then dragInput = input end
    end)
    UserInputService.InputChanged:Connect(function(input)
        if input == dragInput and dragging and not frame:GetAttribute("DragLocked") then
            local delta = input.Position - dragStart
            frame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
        end
    end)
end

local function createUI(cls, props)
    local inst = Instance.new(cls)
    for k, v in pairs(props) do inst[k] = v end
    return inst
end

-- Main Frame
local MainFrame = createUI("Frame", {Parent = ScreenGui, Size = UDim2.new(0, 160, 0, 160), Position = UDim2.new(0.5, -80, 0.5, -80), BackgroundColor3 = Color3.fromRGB(12, 12, 18), Active = true, Visible = false})
createUI("UICorner", {Parent = MainFrame, CornerRadius = UDim.new(0, 8)})
createUI("UIStroke", {Parent = MainFrame, Color = Color3.fromRGB(235, 45, 45), Thickness = 2})
makeDraggable(MainFrame)

local TitleHeader = createUI("Frame", {Parent = MainFrame, Size = UDim2.new(1, 0, 0, 20), BackgroundColor3 = Color3.fromRGB(20, 20, 32)})
createUI("UICorner", {Parent = TitleHeader, CornerRadius = UDim.new(0, 8)})
createUI("TextLabel", {Parent = TitleHeader, Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Text = "⚡ CL90 HUB V1 ⚡", TextColor3 = Color3.fromRGB(45, 140, 255), TextSize = 11, Font = Enum.Font.SourceSansBold})

local TabBar = createUI("Frame", {Parent = MainFrame, Size = UDim2.new(1, -8, 0, 18), Position = UDim2.new(0, 4, 0, 22), BackgroundTransparency = 1, Visible = false})
local MainTabBtn = createUI("TextButton", {Parent = TabBar, Size = UDim2.new(0.48, 0, 1, 0), BackgroundColor3 = Color3.fromRGB(235, 45, 45), Text = "Main", TextColor3 = Color3.fromRGB(255, 255, 255), Font = Enum.Font.SourceSansBold, TextSize = 10})
local SettingsTabBtn = createUI("TextButton", {Parent = TabBar, Size = UDim2.new(0.48, 0, 1, 0), Position = UDim2.new(0.52, 0, 0, 0), BackgroundColor3 = Color3.fromRGB(45, 140, 255), Text = "Settings", TextColor3 = Color3.fromRGB(255, 255, 255), Font = Enum.Font.SourceSansBold, TextSize = 10})
createUI("UICorner", {Parent = MainTabBtn, CornerRadius = UDim.new(0, 4)})
createUI("UICorner", {Parent = SettingsTabBtn, CornerRadius = UDim.new(0, 4)})

local MainPage = createUI("Frame", {Parent = MainFrame, Size = UDim2.new(1, -8, 1, -44), Position = UDim2.new(0, 4, 0, 42), BackgroundTransparency = 1, Visible = false})
local SettingsPage = createUI("Frame", {Parent = MainFrame, Size = UDim2.new(1, -8, 1, -44), Position = UDim2.new(0, 4, 0, 42), BackgroundTransparency = 1, Visible = false})

MainTabBtn.MouseButton1Click:Connect(function() MainPage.Visible = true SettingsPage.Visible = false end)
SettingsTabBtn.MouseButton1Click:Connect(function() MainPage.Visible = false SettingsPage.Visible = true end)

local function makeBtn(parent, text, pos, color, size)
    local btn = createUI("TextButton", {Parent = parent, Size = size or UDim2.new(1, 0, 0, 18), Position = pos, BackgroundColor3 = color, Text = text, TextColor3 = Color3.fromRGB(255, 255, 255), Font = Enum.Font.SourceSansBold, TextSize = 10})
    createUI("UICorner", {Parent = btn, CornerRadius = UDim.new(0, 4)})
    return btn
end

local NormalTeleBtn = makeBtn(MainPage, "Teleport (Behind)", UDim2.new(0, 0, 0, 0), Color3.fromRGB(235, 45, 45))
local FrontTeleBtn = makeBtn(MainPage, "Teleport (In Front)", UDim2.new(0, 0, 0, 20), Color3.fromRGB(45, 140, 255))
local TeleTrackButton = makeBtn(MainPage, "Teleport w/ Track", UDim2.new(0, 0, 0, 40), Color3.fromRGB(235, 45, 45))
local TrackButton = makeBtn(MainPage, "Toggle Track: OFF", UDim2.new(0, 0, 0, 60), Color3.fromRGB(45, 140, 255))
local TeammateBtn = makeBtn(MainPage, "Select Teammates: OFF", UDim2.new(0, 0, 0, 80), Color3.fromRGB(235, 45, 45))
TeammateBtn.TextSize = 9
local TeammateStatus = createUI("TextLabel", {Parent = MainPage, Size = UDim2.new(1, 0, 0, 14), Position = UDim2.new(0, 0, 0, 98), BackgroundTransparency = 1, Text = "Teammates: 0/4", TextColor3 = Color3.fromRGB(170, 170, 185), Font = Enum.Font.SourceSansBold, TextSize = 9})

local RedHighlightBtn = makeBtn(SettingsPage, "ESP Highlights: OFF", UDim2.new(0, 0, 0, 0), Color3.fromRGB(45, 140, 255), UDim2.new(1, 0, 0, 20))
local SeparateBtn = makeBtn(SettingsPage, "Mobile Buttons: OFF", UDim2.new(0, 0, 0, 24), Color3.fromRGB(235, 45, 45), UDim2.new(1, 0, 0, 20))
local TeamResetBtn = makeBtn(SettingsPage, "Team Reset", UDim2.new(0, 0, 0, 48), Color3.fromRGB(45, 140, 255), UDim2.new(1, 0, 0, 20))
local JoinDcBtn = makeBtn(SettingsPage, "Join DC", UDim2.new(0, 0, 0, 72), Color3.fromRGB(235, 45, 45), UDim2.new(1, 0, 0, 20))

-- Discord Notification
local DcNotice = createUI("Frame", {Parent = ScreenGui, Size = UDim2.new(0, 220, 0, 30), Position = UDim2.new(0.5, -110, 0.05, 0), BackgroundColor3 = Color3.fromRGB(12, 12, 18), Visible = false, ZIndex = 40})
createUI("UICorner", {Parent = DcNotice, CornerRadius = UDim.new(0, 6)})
createUI("UIStroke", {Parent = DcNotice, Color = Color3.fromRGB(80, 255, 120), Thickness = 2})
createUI("TextLabel", {Parent = DcNotice, Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Text = "You have copied the Discord link!", TextColor3 = Color3.fromRGB(255, 255, 255), Font = Enum.Font.SourceSansBold, TextSize = 11, ZIndex = 41})

local noticeThread
JoinDcBtn.MouseButton1Click:Connect(function()
    pcall(function() setclipboard("https://discord.gg/JRcZUrKNNz") end)
    DcNotice.Visible = true
    if noticeThread then task.cancel(noticeThread) end
    noticeThread = task.delay(3, function()
        DcNotice.Visible = false
    end)
end)

-- Key System (Key: CL90HUB)
local BlurEffect = createUI("BlurEffect", {Parent = Lighting, Size = 24})
local LockScreen = createUI("Frame", {Parent = ScreenGui, Size = UDim2.new(0, 160, 0, 160), Position = UDim2.new(0.5, -80, 0.5, -80), BackgroundColor3 = Color3.fromRGB(12, 12, 18), ZIndex = 30})
createUI("UICorner", {Parent = LockScreen, CornerRadius = UDim.new(0, 8)})
createUI("UIStroke", {Parent = LockScreen, Color = Color3.fromRGB(235, 45, 45), Thickness = 2})

local KeyHeader = createUI("Frame", {Parent = LockScreen, Size = UDim2.new(1, 0, 0, 24), BackgroundColor3 = Color3.fromRGB(20, 20, 32), ZIndex = 31})
createUI("UICorner", {Parent = KeyHeader, CornerRadius = UDim.new(0, 8)})
createUI("TextLabel", {Parent = KeyHeader, Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Text = "🔒 KEY VERIFICATION", TextColor3 = Color3.fromRGB(45, 140, 255), Font = Enum.Font.SourceSansBold, TextSize = 11, ZIndex = 32})
createUI("TextLabel", {Parent = LockScreen, Size = UDim2.new(1, -16, 0, 16), Position = UDim2.new(0, 8, 0, 28), BackgroundTransparency = 1, Text = "Enter key to unlock CL90 HUB", TextColor3 = Color3.fromRGB(170, 170, 185), Font = Enum.Font.SourceSans, TextSize = 9, ZIndex = 31})

local BoxContainer = createUI("Frame", {Parent = LockScreen, Size = UDim2.new(1, -16, 0, 24), Position = UDim2.new(0, 8, 0, 48), BackgroundColor3 = Color3.fromRGB(22, 22, 32), ZIndex = 31})
createUI("UICorner", {Parent = BoxContainer, CornerRadius = UDim.new(0, 6)})
createUI("UIStroke", {Parent = BoxContainer, Color = Color3.fromRGB(45, 140, 255), Thickness = 1})

local KeyBox = createUI("TextBox", {Parent = BoxContainer, Size = UDim2.new(1, -8, 1, 0), Position = UDim2.new(0, 4, 0, 0), BackgroundTransparency = 1, Text = "", PlaceholderText = "Type key here...", TextColor3 = Color3.fromRGB(255, 255, 255), Font = Enum.Font.SourceSansBold, TextSize = 11, ZIndex = 32})
local SubmitBtn = makeBtn(LockScreen, "UNLOCK SCRIPT", UDim2.new(0, 8, 0, 78), Color3.fromRGB(235, 45, 45), UDim2.new(1, -16, 0, 22))
SubmitBtn.ZIndex = 31

local StatusLabel = createUI("TextLabel", {Parent = LockScreen, Size = UDim2.new(1, -16, 0, 30), Position = UDim2.new(0, 8, 0, 104), BackgroundTransparency = 1, Text = "", TextColor3 = Color3.fromRGB(255, 80, 80), Font = Enum.Font.SourceSansBold, TextSize = 9, TextWrapped = true, ZIndex = 31})
local wrongAttempts = 0

SubmitBtn.MouseButton1Click:Connect(function()
    if KeyBox.Text == "CL90HUB" then
        StatusLabel.TextColor3 = Color3.fromRGB(80, 255, 120)
        StatusLabel.Text = "Access Granted! Loading..."
        task.wait(0.4)
        if BlurEffect then BlurEffect:Destroy() end
        LockScreen:Destroy()
        MainFrame.Visible, TabBar.Visible, MainPage.Visible = true, true, true
    else
        wrongAttempts = wrongAttempts + 1
        if wrongAttempts >= 3 then
            if BlurEffect then BlurEffect:Destroy() end
            ScreenGui:Destroy()
        else
            StatusLabel.Text = "Invalid key!\n(" .. (3 - wrongAttempts) .. " attempts remaining)"
        end
    end
end)

-- ESP & Teammates
local selectedTarget, highlightsActive, highlights, teammatesMode, teammates = nil, false, {}, false, {}
local function isTeammate(p) return table.find(teammates, p) ~= nil end
local function removeHighlight(p) if highlights[p] then pcall(function() highlights[p]:Destroy() end) highlights[p] = nil end end

local function applyHighlightToChar(p, character)
    if not character or isTeammate(p) then removeHighlight(p) return end
    if not highlightsActive then return end
    removeHighlight(p)
    local hl = createUI("Highlight", {Parent = character, OutlineColor = Color3.fromRGB(255, 255, 255), DepthMode = Enum.HighlightDepthMode.AlwaysOnTop, FillColor = (selectedTarget == p) and Color3.fromRGB(80, 255, 120) or Color3.fromRGB(235, 45, 45)})
    highlights[p] = hl
end

local function applyHighlightsToAll()
    for _, p in pairs(Players:GetPlayers()) do if p ~= LocalPlayer and p.Character then applyHighlightToChar(p, p.Character) end end
end

local function updateTeammateUI()
    TeammateStatus.Text = "Teammates: " .. #teammates .. "/4"
    applyHighlightsToAll()
end

TeammateBtn.MouseButton1Click:Connect(function() 
    teammatesMode = not teammatesMode 
    TeammateBtn.Text = "Select Teammates: " .. (teammatesMode and "ON" or "OFF")
    updateTeammateUI()
end)

TeamResetBtn.MouseButton1Click:Connect(function()
    teammates = {}
    updateTeammateUI()
end)

for _, p in pairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then p.CharacterAdded:Connect(function(char) task.wait(0.3) if highlightsActive then applyHighlightToChar(p, char) end end) end
end

Players.PlayerAdded:Connect(function(p)
    p.CharacterAdded:Connect(function(char) task.wait(0.3) if highlightsActive then applyHighlightToChar(p, char) end end)
end)

Players.PlayerRemoving:Connect(function(p)
    removeHighlight(p)
    local idx = table.find(teammates, p)
    if idx then table.remove(teammates, idx) updateTeammateUI() end
end)

RedHighlightBtn.MouseButton1Click:Connect(function()
    highlightsActive = not highlightsActive
    RedHighlightBtn.Text = "ESP Highlights: " .. (highlightsActive and "ON" or "OFF")
    if not highlightsActive then for p in pairs(highlights) do removeHighlight(p) end else applyHighlightsToAll() end
end)

LocalPlayer.CharacterAdded:Connect(function(char)
    task.wait(0.3)
    if highlightsActive then applyHighlightsToAll() end
end)

UserInputService.InputBegan:Connect(function(input)
    if input.UserInputType ~= Enum.UserInputType.MouseButton1 and input.UserInputType ~= Enum.UserInputType.Touch then return end
    if not highlightsActive and not teammatesMode then return end
    local mousePos, closestPlayer, shortestDist = UserInputService:GetMouseLocation(), nil, 80

    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and p.Character then
            local hrp = p.Character:FindFirstChild("HumanoidRootPart") or p.Character.PrimaryPart
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            if hrp and hum and hum.Health > 0 then
                local screenPos, onScreen = Camera:WorldToViewportPoint(hrp.Position)
                if onScreen then
                    local dist = (Vector2.new(mousePos.X, mousePos.Y) - Vector2.new(screenPos.X, screenPos.Y)).Magnitude
                    if dist < shortestDist then shortestDist, closestPlayer = dist, p end
                end
            end
        end
    end

    if closestPlayer then
        if teammatesMode then
            if isTeammate(closestPlayer) then
                table.remove(teammates, table.find(teammates, closestPlayer))
            elseif #teammates < 4 then
                table.insert(teammates, closestPlayer)
                if selectedTarget == closestPlayer then selectedTarget = nil end
            end
            updateTeammateUI()
        elseif highlightsActive and not isTeammate(closestPlayer) then
            if selectedTarget and highlights[selectedTarget] then highlights[selectedTarget].FillColor = Color3.fromRGB(235, 45, 45) end
            selectedTarget = closestPlayer
            if highlights[closestPlayer] then highlights[closestPlayer].FillColor = Color3.fromRGB(80, 255, 120) end
        end
    end
end)

-- Teleport & Tracking
local tracking, trackConnection = false, nil

local function getActiveTarget()
    if selectedTarget and selectedTarget.Parent and selectedTarget.Character and not isTeammate(selectedTarget) then
        local hum = selectedTarget.Character:FindFirstChildOfClass("Humanoid")
        if hum and hum.Health > 0 then return selectedTarget end
    end
    local myHrp = LocalPlayer.Character and (LocalPlayer.Character:FindFirstChild("HumanoidRootPart") or LocalPlayer.Character.PrimaryPart)
    if not myHrp then return nil end
    local closest, shortestDist = nil, math.huge
    for _, p in pairs(Players:GetPlayers()) do
        if p ~= LocalPlayer and not isTeammate(p) and p.Character then
            local pHrp = p.Character:FindFirstChild("HumanoidRootPart") or p.Character.PrimaryPart
            local hum = p.Character:FindFirstChildOfClass("Humanoid")
            if pHrp and hum and hum.Health > 0 then
                local dist = (myHrp.Position - pHrp.Position).Magnitude
                if dist < shortestDist then shortestDist, closest = dist, p end
            end
        end
    end
    return closest
end

local function stopTracking()
    if trackConnection then trackConnection:Disconnect() trackConnection = nil end
    local hum = LocalPlayer.Character and LocalPlayer.Character:FindFirstChildOfClass("Humanoid")
    if hum then hum.WalkSpeed = 16 end
end

local function tpToTarget(offset)
    local target = getActiveTarget()
    local char = LocalPlayer.Character
    if target and target.Character and char then
        local myHrp = char:FindFirstChild("HumanoidRootPart") or char.PrimaryPart
        local targetHrp = target.Character:FindFirstChild("HumanoidRootPart") or target.Character.PrimaryPart
        if myHrp and targetHrp then myHrp.CFrame = targetHrp.CFrame * offset end
    end
end

local function runTeleportTrack()
    tpToTarget(CFrame.new(0, 0, 3))
    stopTracking()
    local startTime = tick()
    trackConnection = RunService.RenderStepped:Connect(function()
        local char = LocalPlayer.Character
        local hum = char and char:FindFirstChildOfClass("Humanoid")
        local target = getActiveTarget()
        if tick() - startTime >= 1.3 or not hum or not target or not target.Character then stopTracking() return end
        hum.WalkSpeed = 18
        local tHrp = target.Character:FindFirstChild("HumanoidRootPart") or target.Character.PrimaryPart
        if tHrp then hum:MoveTo(tHrp.Position) end
    end)
end

NormalTeleBtn.MouseButton1Click:Connect(function() tpToTarget(CFrame.new(0, 0, 3)) end)
FrontTeleBtn.MouseButton1Click:Connect(function() tpToTarget(CFrame.new(0, 0, -3) * CFrame.Angles(0, math.pi, 0)) end)
TeleTrackButton.MouseButton1Click:Connect(runTeleportTrack)

local function toggleTrack()
    tracking = not tracking
    TrackButton.Text = "Toggle Track: " .. (tracking and "ON" or "OFF")
    if tracking then
        trackConnection = RunService.RenderStepped:Connect(function()
            local char = LocalPlayer.Character
            local hum = char and char:FindFirstChildOfClass("Humanoid")
            local target = getActiveTarget()
            if hum and target and target.Character and tracking then
                hum.WalkSpeed = 18
                local tHrp = target.Character:FindFirstChild("HumanoidRootPart") or target.Character.PrimaryPart
                if tHrp then hum:MoveTo(tHrp.Position) end
            else stopTracking() end
        end)
    else stopTracking() end
end
TrackButton.MouseButton1Click:Connect(toggleTrack)

-- Mobile Pills
local separateActive, createdPills = false, {}
local function createAZButton(text, defaultPos, strokeColor, callback)
    local pill = createUI("Frame", {Parent = ScreenGui, Size = UDim2.new(0, 150, 0, 34), Position = defaultPos, BackgroundColor3 = Color3.fromRGB(12, 12, 18), Active = true})
    createUI("UICorner", {Parent = pill, CornerRadius = UDim.new(0, 8)})
    local pillStroke = createUI("UIStroke", {Parent = pill, Color = strokeColor, Thickness = 2})
    local btn = createUI("TextButton", {Parent = pill, Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Text = text, TextColor3 = Color3.fromRGB(255, 255, 255), Font = Enum.Font.SourceSansBold, TextSize = 10})

    local cooldownThread
    makeDraggable(pill, btn, function()
        if pill:GetAttribute("DragLocked") then return end
        if cooldownThread then task.cancel(cooldownThread) end
        cooldownThread = task.delay(5, function() pill:SetAttribute("DragLocked", true) pillStroke.Transparency = 0.5 end)
    end)

    local startPos, dragDistance = nil, 0
    btn.InputBegan:Connect(function(input) if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then startPos, dragDistance = input.Position, 0 end end)
    btn.InputChanged:Connect(function(input) if startPos then dragDistance = (input.Position - startPos).Magnitude end end)
    btn.InputEnded:Connect(function(input) if dragDistance < 10 then callback() end startPos = nil end)

    table.insert(createdPills, pill)
    return btn
end

SeparateBtn.MouseButton1Click:Connect(function()
    separateActive = not separateActive
    SeparateBtn.Text = "Mobile Buttons: " .. (separateActive and "ON" or "OFF")
    if separateActive then
        createAZButton("CL90: TELEPORT (BEHIND)", UDim2.new(0.05, 0, 0.27, 0), Color3.fromRGB(235, 45, 45), function() tpToTarget(CFrame.new(0, 0, 3)) end)
        createAZButton("CL90: TELEPORT (FRONT)", UDim2.new(0.05, 0, 0.35, 0), Color3.fromRGB(45, 140, 255), function() tpToTarget(CFrame.new(0, 0, -3) * CFrame.Angles(0, math.pi, 0)) end)
        createAZButton("CL90: TELEPORT W/ TRACK", UDim2.new(0.05, 0, 0.43, 0), Color3.fromRGB(235, 45, 45), runTeleportTrack)
        createAZButton("CL90: TRACK TOGGLE", UDim2.new(0.05, 0, 0.51, 0), Color3.fromRGB(45, 140, 255), toggleTrack)
    else
        for _, pill in pairs(createdPills) do pcall(function() pill:Destroy() end) end
        createdPills = {}
    end
end)
