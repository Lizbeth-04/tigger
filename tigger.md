1)Validar que el número de cédula del cliente tenga 10 números (no letras)
CREATE OR REPLACE FUNCTION validate_nui()
RETURNS TRIGGER AS $$
BEGIN
    IF LENGTH(NEW.nui) != 10 OR NEW.nui ~ '[^0-9]' THEN
        RAISE EXCEPTION 
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
 *crear tigger:
 CREATE TRIGGER validate_nui_trigger
BEFORE INSERT OR UPDATE ON client
FOR EACH ROW
EXECUTE PROCEDURE validate_nui();

2)Disminuir el stock de la tabla product cuando se inserte un nuevo registro en la tabla item
 CREATE OR REPLACE FUNCTION decrease_product_stock()
RETURNS TRIGGER AS $$
BEGIN
    UPDATE product
    SET stock = stock - NEW.quantity
    WHERE id = NEW.product_id;

    IF NOT FOUND THEN
        RAISE EXCEPTION 'Producto no encontrado';
    END IF;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
*crear tigger:
CREATE TRIGGER decrease_stock_trigger
AFTER INSERT ON item
FOR EACH ROW
EXECUTE PROCEDURE decrease_product_stock();

3)Validar que el campo create_at de la tabla invoice sea del año actual
CREATE OR REPLACE FUNCTION validate_create_at()
RETURNS TRIGGER AS $$
BEGIN
    IF EXTRACT(YEAR FROM NEW.create_at) != EXTRACT(YEAR FROM CURRENT_DATE) THEN
        RAISE EXCEPTION 'La fecha de creación debe ser del año actual';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
*crear tigger:
CREATE TRIGGER validate_create_at_trigger
BEFORE INSERT OR UPDATE ON invoice
FOR EACH ROW
EXECUTE PROCEDURE validate_create_at();

4)4. Validar que el correo del cliente tenga un @
CREATE OR REPLACE FUNCTION validate_email()
RETURNS TRIGGER AS $$
BEGIN
    IF POSITION('@' IN NEW.email) = 0 THEN
        RAISE EXCEPTION 'El correo electrónico debe contener un @';
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;
*crear tigger:
CREATE TRIGGER validate_email_trigger
BEFORE INSERT OR UPDATE ON client
FOR EACH ROW
EXECUTE PROCEDURE validate_email();
