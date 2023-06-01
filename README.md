import baselayer
import layer2

class InvalidOrdinalError(Exception):
    """Error raised when an invalid ordinal is passed to the function."""

    def __init__(self, ordinal):
        super().__init__("The ordinal must be a positive integer.")
        self.ordinal = ordinal

class TransactionFailedError(Exception):
    """Error raised when a transaction fails."""

    def __init__(self, error_message):
        super().__init__("The transaction failed with the following error: {}".format(error_message))
        self.error_message = error_message

def validate_ordinals(ordinals):
    """Validates a list of ordinals.

    Args:
      ordinals: A list of ordinals to validate.

    Raises:
      InvalidOrdinalError: If an invalid ordinal is found.
    """
    for ordinal in ordinals:
        if not isinstance(ordinal, int) or ordinal < 0:
            raise InvalidOrdinalError(ordinal)

def move_ordinals_to_layer2(ordinals):
    """Moves the given ordinals to layer 2.

    Args:
      ordinals: A list of ordinals to move.

    Returns:
      A list of the ordinals that were moved.
    """
    validate_ordinals(ordinals)

    # Get the layer 2 connection.
    connection = layer2.get_connection()

    # Create a transaction to move the ordinals.
    transaction = layer2.Transaction()
    for ordinal in ordinals:
        transaction.add_ordinal(ordinal)

    # Sign the transaction.
    private_key = baselayer.get_private_key()
    if private_key is None:
        raise ValueError("Private key is missing or inaccessible")
    transaction.sign(private_key)

    # Submit the transaction.
    try:
        connection.submit_transaction(transaction)
    except layer2.ConnectionError as e:
        raise TransactionFailedError("Transaction submission failed") from e

    # Get the results of the transaction.
    try:
        results = connection.get_transaction_results(transaction.id)
    except layer2.ConnectionError as e:
        raise TransactionFailedError("Failed to retrieve transaction results") from e

    # Check if the transaction was successful.
    if results.status != layer2.TransactionStatus.SUCCESS:
        raise TransactionFailedError("Transaction failed")

    return results.ordinals

if __name__ == "__main__":
    # Get the ordinals to move.
    ordinals = [1, 2, 3]

    try:
        # Move the ordinals to layer 2.
        moved_ordinals = move_ordinals_to_layer2(ordinals)

        # Print the moved ordinals.
        for ordinal in moved_ordinals:
            print(ordinal)
    except (InvalidOrdinalError, TransactionFailedError) as e:
        print("Error:", str(e))
