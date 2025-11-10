-- Inspector para Pasarela: lista estructura y propiedades útiles
-- Ejecuta esto en el mismo lugar donde probaste el collector.
local CollectionService = game:GetService("CollectionService")
local Players = game:GetService("Players")

local nombrePasarela = "Pasarela" -- cambia esto si la carpeta tiene otro nombre
local maxChildrenToShow = 200

local function safePrint(...)
    local ok, _ = pcall(function() print(...) end)
    -- ignorar fallos de print en algunos ejecutores
end

local function isPlayerModel(model)
    if not model or not model:IsA("Model") then return false end
    for _, p in ipairs(Players:GetPlayers()) do
        if p.Character == model then return true end
    end
    return false
end

local function inspectModel(m)
    safePrint("---- Modelo:", m:GetFullName())
    safePrint(" Name:", m.Name, " Class:", m.ClassName)
    safePrint(" IsDescendantOf workspace:", tostring(m:IsDescendantOf(workspace)))
    safePrint(" IsA Model:", tostring(m:IsA("Model")))
    safePrint(" IsPlayerCharacter:", tostring(isPlayerModel(m)))
    local humanoid = m:FindFirstChildWhichIsA("Humanoid")
    safePrint(" Has Humanoid:", tostring(not not humanoid))
    local hrp = m:FindFirstChild("HumanoidRootPart") or m.PrimaryPart
    safePrint(" Has HRP/PrimaryPart:", tostring(not not hrp))
    -- CollectionService tags (pcall por seguridad)
    local ok, tags = pcall(function() return CollectionService:GetTags(m) end)
    if ok and tags and #tags > 0 then
        safePrint(" CollectionService Tags:", table.concat(tags, ", "))
    else
        safePrint(" CollectionService Tags: none or inaccessible")
    end
    -- listar hijos (StringValue / BoolValue / NumberValue / ObjectValues)
    for _, child in ipairs(m:GetChildren()) do
        local t = child.ClassName
        if t == "StringValue" or t == "BoolValue" or t == "IntValue" or t == "NumberValue" or t == "ObjectValue" then
            local val = nil
            pcall(function() val = child.Value end)
            safePrint("  -> Child:", child.Name, "Class:", t, "Value:", tostring(val))
        else
            safePrint("  -> Child:", child.Name, "Class:", t)
        end
    end
    safePrint("---- FIN modelo ----")
end

-- START
safePrint("=== INSPECTOR: buscando contenedor '" .. nombrePasarela .. "' en workspace ===")
local pasarela = workspace:FindFirstChild(nombrePasarela)
if not pasarela then
    safePrint("[INSPECTOR] No se encontró con FindFirstChild('"..nombrePasarela.."'). Intentando WaitForChild 5s...")
    local ok, res = pcall(function() return workspace:WaitForChild(nombrePasarela, 5) end)
    if ok and res then
        pasarela = res
        safePrint("[INSPECTOR] Pasarela encontrada con WaitForChild.")
    else
        safePrint("[INSPECTOR] NO se encontró la pasarela. Lista de hijos de workspace:")
        for i, c in ipairs(workspace:GetChildren()) do
            safePrint(" -", i, c.Name, c.ClassName)
        end
        return
    end
end

safePrint("[INSPECTOR] Pasarela encontrada:", pasarela:GetFullName(), " Hijos:", #pasarela:GetChildren())

local count = 0
for _, child in ipairs(pasarela:GetChildren()) do
    count = count + 1
    if count > maxChildrenToShow then
        safePrint("[INSPECTOR] Límite de hijos alcanzado:", maxChildrenToShow)
        break
    end
    if child:IsA("Model") then
        inspectModel(child)
    else
        safePrint("Elemento no model en pasarela:", child.Name, child.ClassName)
    end
    task.wait(0.03)
end

safePrint("=== INSPECTOR: FIN ===")
